# 🐳 Docker 운영 관리 - Resource, Health Check, Environment & Secret

## 📌 학습 목표

Docker 컨테이너를 실제 서비스 환경에서 운영할 때 필요한 기능들을 학습하고 실습한다.

이번 학습에서는 다음 내용을 다루었다.

- Restart Policy
- Health Check
- CPU / Memory Resource Limit
- OOM과 Exit Code 137
- Logging 제한
- 환경변수 외부 주입
- Docker Secret
- `.gitignore`를 이용한 설정 파일 관리
- `docker inspect`를 이용한 컨테이너 상태 확인

---

# 1. Restart Policy

Restart Policy는 **컨테이너가 종료되었을 때 Docker가 컨테이너를 다시 실행하도록 설정하는 기능**이다.

```yaml
restart: unless-stopped
```

`unless-stopped`를 사용하면 컨테이너가 비정상적으로 종료되었을 때 다시 실행되지만, 사용자가 직접 중지한 경우에는 자동으로 다시 실행하지 않는다.

### 핵심 흐름

```text
Container 실행
      ↓
프로세스 종료
      ↓
Restart Policy 확인
      ↓
자동 재시작
```

재시작 횟수와 상태는 `docker inspect`를 통해 확인할 수 있다.

```bash
docker inspect -f 'RestartCount={{.RestartCount}} Status={{.State.Status}}' myapi-api-1
```

---

# 2. Health Check

Health Check는 **컨테이너 내부의 애플리케이션이 실제로 정상 동작하는지 검사하는 기능**이다.

컨테이너가 `running` 상태라고 해서 내부 애플리케이션까지 정상이라는 의미는 아니다.

```text
Container Running
       ↓
Health Check
       ↓
 healthy / unhealthy
```

### Restart Policy와 차이

| 기능 | 역할 |
|---|---|
| Restart Policy | 종료된 컨테이너를 다시 실행 |
| Health Check | 애플리케이션의 정상 동작 여부 확인 |

Health Check가 실패하여 `unhealthy`가 되어도 **컨테이너가 자동으로 재시작되는 것은 아니다.**

즉,

```text
Health Check = 상태 확인
Restart Policy = 종료 후 재실행
```

으로 구분할 수 있다.

---

# 3. CPU / Memory Resource Limit

컨테이너가 서버의 CPU와 Memory를 무제한으로 사용하면 다른 서비스에 영향을 줄 수 있다.

따라서 운영 환경에서는 컨테이너별 Resource Limit을 설정한다.

현재 사용량은 다음 명령어로 확인할 수 있다.

```bash
docker stats --no-stream
```

실습에서 API 컨테이너의 메모리 사용량은 약 **220~285MiB** 정도였다.

이를 기준으로 순간적인 사용량 증가를 고려하여 Memory Limit을 `512M`으로 설정했다.

```yaml
deploy:
  resources:
    limits:
      cpus: "0.5"
      memory: "512M"
```

적용 후:

```bash
docker compose up -d
docker stats --no-stream
```

확인 결과:

```text
MEM USAGE / LIMIT
225.5MiB / 512MiB
```

Resource Limit이 정상적으로 적용된 것을 확인했다.

### 핵심

```text
실제 Resource 사용량 측정
        ↓
필요한 여유 계산
        ↓
Resource Limit 설정
        ↓
docker stats로 확인
```

---

# 4. OOM과 Exit Code 137

Memory Limit을 실제 필요한 메모리보다 낮게 설정하여 의도적으로 장애를 발생시켰다.

그 결과 API 컨테이너에서 다음 상태를 확인했다.

```text
Restarting (137)
```

Exit Code 137은 다음과 같이 해석할 수 있다.

```text
137 = 128 + 9

Signal 9 = SIGKILL
```

즉 프로세스가 `SIGKILL`에 의해 강제로 종료되었다는 의미이다.

메모리 부족 여부는 추가로 확인한다.

```bash
docker inspect myapi-api-1 | grep OOMKilled
```

```text
Exit Code 137
      +
OOMKilled = true
      ↓
Memory Limit 초과로 인한 OOM
```

### Linux OOM과 Container OOM 차이

| Linux Server | Docker Container |
|---|---|
| 시스템 전체 메모리 부족 | 컨테이너 Memory Limit 초과 |
| 서버 전체에 영향을 줄 수 있음 | 해당 컨테이너 중심으로 영향 |
| 실제 서버 메모리 기준 | 컨테이너에 설정한 Limit 기준 |

Docker에서는 호스트에 메모리가 남아 있어도 **컨테이너의 Memory Limit을 초과하면 OOM이 발생할 수 있다.**

---

# 5. Logging 제한

컨테이너 로그가 무제한으로 저장되면 서버의 디스크 공간을 계속 사용할 수 있다.

따라서 로그 파일의 크기와 개수를 제한한다.

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

설정 의미:

```text
로그 파일 최대 크기 = 10MB
로그 파일 최대 개수 = 3개
```

설정 적용 여부는 다음과 같이 확인한다.

```bash
docker inspect myapi-api-1 --format '{{json .HostConfig.LogConfig}}'
```

---

# 6. 환경변수를 코드 밖으로 분리

환경별 설정값을 Docker 이미지 내부에 고정하지 않고 외부 파일에서 전달할 수 있다.

예시:

```env
APP_ENV=dev
MODEL_NAME=distilbert-base-uncased-finetuned-sst-2-english
```

Compose에서는 다음과 같이 사용한다.

```yaml
env_file:
  - ./app.env
```

환경변수를 변경한 후:

```bash
docker compose up -d
curl localhost:8080/healthz
```

결과:

```json
{"status":"ok","env":"dev"}
```

이를 통해 **같은 Docker Image를 서로 다른 설정으로 실행할 수 있다.**

```text
              동일한 Image
                   │
          ┌────────┴────────┐
          ↓                 ↓
     APP_ENV=dev       APP_ENV=prod
          ↓                 ↓
      개발 환경           운영 환경
```

설정값이 변경되었다고 해서 Docker 이미지를 다시 빌드할 필요는 없다.

---

# 7. Secret 관리

비밀번호나 API Key와 같은 민감한 정보는 일반 설정과 분리하여 관리할 수 있다.

Secret 파일 생성:

```bash
mkdir -p secrets
echo "test-secret-1234" > secrets/api_token.txt
```

Compose에서 Secret을 연결한다.

```yaml
services:
  api:
    secrets:
      - api_token

secrets:
  api_token:
    file: ./secrets/api_token.txt
```

컨테이너 내부에서는 다음 경로에서 읽을 수 있다.

```bash
docker exec myapi-api-1 cat /run/secrets/api_token
```

```text
test-secret-1234
```

구조:

```text
Host

secrets/api_token.txt
        │
        │ Docker Compose
        ↓
Container

/run/secrets/api_token
```

---

# 8. 환경변수와 Secret 비교

`docker inspect`는 컨테이너의 상세 설정과 상태를 조회하는 명령어이다.

```bash
docker inspect myapi-api-1
```

환경변수는 컨테이너 설정에서 확인될 수 있다.

```bash
docker inspect myapi-api-1 | grep APP_ENV
```

반면 Secret은 파일로 전달되기 때문에 실제 Secret 값이 `docker inspect`에 직접 노출되지 않는 것을 확인할 수 있다.

```text
환경변수
APP_ENV=dev
    ↓
docker inspect에서 확인 가능

Secret
test-secret-1234
    ↓
파일로 전달
    ↓
실제 값은 inspect에 직접 노출되지 않음
```

---

# 9. .gitignore를 이용한 설정 파일 보호

환경설정 파일이나 Secret 파일에는 민감한 정보가 포함될 수 있기 때문에 Git 저장소에 올리지 않도록 관리해야 한다.

`.gitignore`

```gitignore
app.env
dev.env
secrets/
```

구조는 다음과 같이 관리할 수 있다.

```text
Project
│
├── app.py                 ✅ Git
├── Dockerfile             ✅ Git
├── compose.yaml           ✅ Git
├── requirements.txt       ✅ Git
│
├── app.env                ❌ Git 제외
├── dev.env                ❌ Git 제외
├── secrets/               ❌ Git 제외
│
├── app.env.example        ✅ Git
└── .gitignore             ✅ Git
```

실제 설정값과 Secret은 저장소에서 제외하지만 **다른 개발자가 어떤 설정값이 필요한지 알 수 있도록 예시 파일은 저장소에 포함한다.**

> 실제 값은 숨기고, 필요한 설정 항목은 Example 파일을 통해 공유한다.

---

# 10. Troubleshooting - Memory Limit / OOM

## 증상

Memory Limit을 실제 사용량보다 낮게 설정한 후 API 컨테이너가 정상적으로 실행되지 않았다.

```text
Restarting (137)
```

## 확인 과정

```bash
docker ps
docker stats
docker inspect
```

## 원인

API 프로세스가 필요로 하는 메모리보다 낮은 Memory Limit이 설정되어 OOM이 발생했다.

```text
Memory Limit 초과
        ↓
OOM
        ↓
SIGKILL
        ↓
Exit Code 137
        ↓
Restart Policy
        ↓
Container 재시작
```

## 해결

실제 메모리 사용량을 측정한 뒤 여유를 고려하여 Memory Limit을 `512MiB`로 변경했다.

## 해결 확인

```bash
docker compose up -d
docker compose ps
docker stats --no-stream
```

컨테이너가 `healthy` 상태로 실행되고 Memory Limit도 정상적으로 적용된 것을 확인했다.

---

# 📌 핵심 정리

이번 학습에서는 Docker 컨테이너를 **실행하는 방법에서 한 단계 더 나아가 운영하는 방법**을 학습했다.

```text
Container 실행
      ↓
Health Check
      ↓
Restart Policy
      ↓
Resource Limit
      ↓
Logging
      ↓
Environment
      ↓
Secret
      ↓
Troubleshooting
```

특히 다음 내용을 중요하게 이해했다.

- `running`과 `healthy`는 다른 개념이다.
- Health Check와 Restart Policy는 역할이 다르다.
- 컨테이너에는 CPU와 Memory Limit을 설정할 수 있다.
- Exit Code 137은 SIGKILL에 의한 종료를 의미한다.
- OOM 여부는 `docker inspect` 등을 통해 추가 확인해야 한다.
- 로그가 무제한으로 쌓이지 않도록 크기와 개수를 제한할 수 있다.
- 환경설정을 이미지와 분리하면 같은 이미지를 여러 환경에서 사용할 수 있다.
- 비밀번호와 API Key 같은 정보는 Secret으로 분리하여 관리할 수 있다.
- 실제 환경설정과 Secret은 `.gitignore`를 이용해 Git 저장소에서 제외한다.
- 장애 발생 시 상태 → 로그/자원 → 설정 → 원인 → 해결 → 검증 순서로 접근한다.

Docker에서 학습한 **Health Check, Resource Limit, 환경설정, Secret, 장애 복구** 등의 개념은 이후 Kubernetes 환경에서도 계속 확장되어 사용된다.
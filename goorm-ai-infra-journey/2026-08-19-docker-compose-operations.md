# Docker Compose 운영 기초

## 🎯 오늘 배운 핵심

```text
env_file       → 일반 설정값 관리
secrets        → 비밀번호/API Key 등 민감정보 관리
healthcheck    → 서비스가 실제 정상인지 확인
restart        → 컨테이너 프로세스 종료 시 재시작
depends_on     → 서비스 간 의존 조건 설정
resource limit → CPU / Memory 과사용 방지
logging        → 로그가 무한정 쌓이는 것 방지
```

> 핵심: 컨테이너를 실행하는 것뿐 아니라 **상태·자원·보안·로그까지 관리하는 것이 운영의 기본**이다.

---

## 1. 환경변수와 Secret

일반 설정은 `env_file`, 민감정보는 `secrets`로 분리한다.

```yaml
env_file:
  - ./app.env

secrets:
  - api_token
```

Secret은 컨테이너 내부에서 파일로 확인할 수 있다.

```bash
docker compose exec api cat /run/secrets/api_token
```

민감한 파일은 Git에 올리지 않는다.

```gitignore
*.env
secrets/
```

---

## 2. Healthcheck / Restart / 의존성

### Healthcheck
서비스가 **실제로 정상 동작하는지** 검사한다.

```text
Running ≠ Healthy
```

특히 중요:

```text
unhealthy
→ 상태가 안 좋다는 표시
→ 이것만으로 restart 되는 것은 아님

프로세스 종료
→ 컨테이너 종료
→ Restart Policy 동작
```

Restart 설정:

```yaml
restart: unless-stopped
```

API가 `healthy`가 된 후 Proxy가 의존하도록 설정:

```yaml
depends_on:
  api:
    condition: service_healthy
```

---

## 3. CPU / Memory 제한

한 컨테이너가 Host 자원을 과도하게 사용하는 것을 막는다.

```yaml
resources:
  limits:
    cpus: "1.5"
    memory: "2G"
```

현재 사용량 확인:

```bash
docker stats
```

메모리 제한을 초과하면 OOM으로 프로세스가 종료될 수 있다.

---

## 4. Log Rotation

로그를 무제한 쌓아두면 디스크를 차지하므로 크기와 개수를 제한한다.

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

```bash
docker compose logs -f api
```

---

## 5. Docker Host 차이

Linux는 Docker Engine이 Host에서 직접 동작한다.

```text
Linux → Docker → Container
```

Mac Docker Desktop은 중간에 Linux VM이 존재한다.

```text
macOS → Docker Desktop → Linux VM → Container
```

따라서 AWS EC2 같은 Linux 서버에서 Docker를 사용하면 Host와 Container 구조를 더 직접적으로 확인할 수 있다.

---

## ⭐ 기억할 명령어

```bash
docker compose up -d       # 실행
docker compose ps          # 상태 확인
docker compose logs -f api # 실시간 로그
docker compose config -q   # compose 문법 검사

docker stats               # CPU / Memory 사용량
docker inspect <container> # 컨테이너 상세 설정
docker exec -it <container> sh  # 컨테이너 내부 진입
```

## 한 줄 정리

**Compose 문법을 외우기보다 `왜 이 설정이 필요한가?`를 이해하는 것이 핵심이다.**
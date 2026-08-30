# Kubernetes TIL - Service, Ingress, ConfigMap, Secret

> 오늘의 핵심:  
> **Pod를 외부와 연결하고(Service/Ingress), 애플리케이션의 설정을 이미지에서 분리한다(ConfigMap/Secret).**

---

# 1. Kubernetes Service

Pod는 생성/삭제될 수 있고 IP도 바뀔 수 있기 때문에  
Pod에 직접 접근하기보다는 **Service를 앞에 둔다.**

```text
사용자
  ↓
Service
  ↓
Pod
```

Service는 selector를 이용해 연결할 Pod를 찾는다.

```yaml
spec:
  selector:
    app: web
```

즉,

```text
Service
   ↓ selector: app=web
   ↓
app=web 라벨을 가진 Pod
```

## Service 종류

| 종류 | 용도 |
|---|---|
| ClusterIP | 클러스터 내부 통신 |
| NodePort | 노드의 포트를 열어 외부 접근 |
| LoadBalancer | 클라우드의 Load Balancer를 이용해 외부 공개 |

### port와 targetPort

```yaml
ports:
  - port: 80
    targetPort: 8000
```

의미:

```text
Service :80
     ↓
Pod :8000
```

- `port` : Service가 받는 포트
- `targetPort` : 실제 컨테이너가 사용하는 포트

---

# 2. Kubernetes Ingress

Service가 Pod 앞에 있다면,  
Ingress는 **여러 Service 앞에서 요청을 어디로 보낼지 결정**한다.

```text
사용자
   ↓
Ingress Controller
   ↓
Ingress 규칙 확인
   ↓
Service
   ↓
Pod
```

핵심 역할을 구분하면:

```text
Ingress → 어느 Service로 보낼까?
Service → 어느 Pod로 보낼까?
```

## 경로(Path)에 따른 라우팅

예:

```text
/       → web Service
/blue   → blue Service
```

전체 흐름:

```text
localhost:8080/blue
        ↓
Ingress Controller
        ↓
"/blue는 blue Service"
        ↓
blue Service
        ↓
blue Pod
        ↓
BLUE
```

실습에서 실제로 다음과 같이 확인했다.

```bash
curl -s localhost:8080/
```

→ nginx Web 페이지

```bash
curl -s localhost:8080/blue
```

→ `BLUE`

## Ingress와 Ingress Controller

둘은 다르다.

```text
Ingress
= 라우팅 규칙

Ingress Controller
= 그 규칙을 실제로 실행하는 프로그램
```

Minikube에서는 nginx ingress controller를 사용했다.

```bash
minikube addons enable ingress
```

Controller 확인:

```bash
kubectl get pods -n ingress-nginx
```

---

# 3. ConfigMap

애플리케이션의 **설정값을 이미지와 분리**하기 위해 사용한다.

예를 들어 이미지 안에 다음 설정을 직접 넣어버리면:

```text
LOG_LEVEL=debug
BATCH_SIZE=16
```

운영 환경마다 이미지를 다시 만들어야 할 수 있다.

ConfigMap을 사용하면:

```text
            같은 이미지
          sentiment:2
               │
       ┌───────┴───────┐
       ↓               ↓
     dev Pod         prod Pod
       ↓               ↓
   ConfigMap         ConfigMap
 LOG=debug          LOG=warn
 BATCH=16           BATCH=64
```

즉,

> **코드는 한 벌, 설정은 여러 벌**

이라는 구조를 만들 수 있다.

---

# 4. ConfigMap 생성

실습에서 사용한 dev ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config
  namespace: dev

data:
  MODEL_NAME: "sentiment"
  LOG_LEVEL: "debug"
  BATCH_SIZE: "16"
```

적용:

```bash
kubectl apply -f 52-app-config.yaml
```

확인:

```bash
kubectl get cm -n dev
```

상세 확인:

```bash
kubectl describe cm app-config -n dev
```

---

# 5. ConfigMap 값을 환경변수로 사용

ConfigMap을 만들었다고 Pod가 자동으로 사용하는 것은 아니다.

Pod에서 ConfigMap을 **참조**해야 한다.

## env + configMapKeyRef

특정 값을 하나씩 가져오는 방법이다.

```yaml
env:
  - name: MODEL_NAME
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: MODEL_NAME

  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

구조:

```text
ConfigMap
app-config
   │
   ├── MODEL_NAME
   ├── LOG_LEVEL
   └── BATCH_SIZE
          ↓
     필요한 값 선택
          ↓
     Pod 환경변수
```

Pod 안에서 확인:

```bash
kubectl exec app -n dev -- env | grep -E "MODEL|LOG|BATCH"
```

실습 결과:

```text
MODEL_NAME=sentiment
LOG_LEVEL=debug
BATCH_SIZE=16
```

---

# 6. 같은 YAML을 dev/prod에서 사용

실습에서 `53-app-env.yaml`에는 namespace를 넣지 않았다.

그래서 적용할 때 namespace를 지정했다.

```bash
kubectl apply -f 53-app-env.yaml -n dev
```

그리고 같은 파일을 prod에도 적용했다.

```bash
kubectl apply -f 53-app-env.yaml -n prod
```

중요한 점:

```text
dev/app-config
        ≠
prod/app-config
```

이름은 둘 다 `app-config`여도  
**namespace가 다르면 서로 다른 Kubernetes 리소스**다.

실습 결과:

```text
dev
LOG_LEVEL=debug
BATCH_SIZE=16

prod
LOG_LEVEL=warn
BATCH_SIZE=64
```

Pod YAML은 같지만 ConfigMap이 다르기 때문에  
환경별로 다른 설정을 사용할 수 있었다.

---

# 7. ConfigMap을 변경하면 Pod도 바로 바뀔까?

환경변수로 ConfigMap을 넣은 경우 **바로 바뀌지 않는다.**

처음 상태:

```text
ConfigMap = debug
Pod       = debug
```

ConfigMap을 수정:

```yaml
LOG_LEVEL: "info"
```

그리고 다시 적용:

```bash
kubectl apply -f 52-app-config.yaml
```

ConfigMap을 확인하면:

```bash
kubectl get cm app-config -n dev -o yaml | grep LOG
```

결과:

```text
LOG_LEVEL: info
```

하지만 기존 Pod를 확인하면:

```bash
kubectl exec app -n dev -- env | grep LOG
```

여전히:

```text
LOG_LEVEL=debug
```

였다.

## 이유

환경변수는 **컨테이너 프로세스가 시작될 때 읽기 때문**이다.

```text
Pod 시작
   ↓
ConfigMap 읽음
   ↓
LOG_LEVEL=debug
   ↓
컨테이너 실행
   ↓
ConfigMap을 info로 변경
   ↓
기존 환경변수는 여전히 debug
```

따라서 새 값을 적용하려면 Pod를 다시 만들어야 한다.

실습에서는:

```bash
kubectl delete -f 53-app-env.yaml -n dev
kubectl apply -f 53-app-env.yaml -n dev
```

이후 확인:

```bash
kubectl exec app -n dev -- env | grep LOG
```

결과:

```text
LOG_LEVEL=info
```

Deployment라면 상황에 따라 다음과 같은 방식으로 새 Pod를 만들 수 있다.

```bash
kubectl rollout restart deployment/<deployment-name>
```

### 기억할 것

> ConfigMap을 환경변수로 사용했다면  
> **ConfigMap 변경 = 기존 Pod 환경변수 자동 변경이 아니다.**

---

# 8. env와 envFrom 차이

ConfigMap의 키가 많아지면 하나씩 작성하기 불편하다.

## env

필요한 값을 하나씩 선택한다.

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

장점:

```text
필요한 값만 선택 가능
환경변수 이름을 다르게 지정 가능
```

## envFrom

ConfigMap 전체를 환경변수로 가져온다.

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

ConfigMap이:

```yaml
data:
  MODEL_NAME: "sentiment"
  LOG_LEVEL: "debug"
  BATCH_SIZE: "16"
```

라면 Pod에는:

```text
MODEL_NAME=sentiment
LOG_LEVEL=debug
BATCH_SIZE=16
```

로 들어간다.

### 한 줄 정리

```text
env     = 필요한 키를 하나씩
envFrom = ConfigMap을 통째로
```

---

# 9. Docker Compose env_file과 연결

Docker Compose에서 사용했던:

```yaml
env_file:
  - ./app.env
```

와 Kubernetes ConfigMap은 비슷한 목적을 가진다.

Docker Compose:

```text
app.env
   ↓
env_file
   ↓
Container 환경변수
```

Kubernetes:

```text
ConfigMap
   ↓
env / envFrom
   ↓
Pod 환경변수
```

차이점은 Compose의 `env_file`은 파일을 사용하고,  
Kubernetes의 ConfigMap은 설정을 **클러스터의 리소스로 등록해서 관리**할 수 있다는 것이다.

---

# 10. ConfigMap을 파일로 넣기

모든 프로그램이 환경변수로 설정을 받는 것은 아니다.

예를 들어 어떤 프로그램은:

```text
/etc/app/config.yaml
```

같은 설정 파일을 필요로 할 수 있다.

이 경우 ConfigMap을 **파일 형태로 Pod에 마운트**할 수 있다.

핵심 구조:

```text
ConfigMap
    ↓
volume
    ↓
volumeMount
    ↓
Container 내부 파일
```

여기서:

```text
volumes
→ 무엇을 마운트할지 정의

volumeMounts
→ 컨테이너의 어디에 마운트할지 정의
```

즉 ConfigMap은:

```text
ConfigMap
   │
   ├── env / envFrom
   │       ↓
   │    환경변수
   │
   └── volumes + volumeMounts
           ↓
          파일
```

두 가지 방식으로 사용할 수 있다.

---

# 11. ConfigMap의 제한

ConfigMap은 모든 데이터를 넣는 저장소가 아니다.

대표적으로 기억할 점:

```text
크기 제한 → ConfigMap 하나당 최대 1MiB
범위      → namespace 단위
용도      → 설정값
```

따라서 대용량 모델 파일이나 큰 데이터 파일을 ConfigMap에 넣는 용도로 사용하면 안 된다.

또한 ConfigMap은 **민감한 정보를 저장하기 위한 리소스가 아니다.**

---

# 12. Secret

비밀번호, API Key, Token 같은 민감한 값은  
ConfigMap과 분리해서 **Secret**으로 관리한다.

```text
일반 설정                 민감한 정보

LOG_LEVEL=debug            DB_PASSWORD
BATCH_SIZE=16              API_KEY
MODEL_NAME=sentiment       TOKEN
       ↓                       ↓
   ConfigMap                 Secret
```

따라서 기본적인 역할 구분은:

```text
ConfigMap
→ 일반 설정

Secret
→ 민감한 설정
```

Secret 역시 Pod에 환경변수나 파일 등의 형태로 전달할 수 있다.

---

# 13. Secret이라고 무조건 안전한 것은 아니다

Kubernetes Secret의 값을 YAML 등에서 볼 때 Base64로 표현되는 경우가 있다.

예:

```text
password123
     ↓
Base64
     ↓
cGFzc3dvcmQxMjM=
```

하지만 **Base64는 암호화가 아니다.**

누구든 값을 다시 디코딩할 수 있다.

따라서:

> Secret이라는 이름만 보고 완벽한 암호화 저장소라고 생각하면 안 된다.

실제 운영에서는 Secret 자체뿐만 아니라:

- Kubernetes 접근 권한
- RBAC
- 저장 데이터 암호화
- 외부 Secret 관리 시스템

등의 보안도 함께 고려해야 한다.

---

# 14. 전체 구조

오늘 배운 내용을 하나의 그림으로 정리하면:

```text
                   Kubernetes
                       │
        ┌──────────────┴──────────────┐
        │                             │
      네트워크                       설정
        │                             │
   Service / Ingress         ┌────────┴────────┐
        │                    │                 │
        │                일반 설정           비밀 정보
        │                    │                 │
        │                ConfigMap           Secret
        │                    │                 │
        │              ┌─────┴─────┐     ┌────┴────┐
        │              │           │     │         │
        │          환경변수        파일   환경변수    파일
        │          env/envFrom     │
        │                          │
        │                  volume/volumeMount
        │
        ▼
       Pod
```

---

# 오늘 배운 핵심

### Service / Ingress

```text
Ingress → 어느 Service로?
Service → 어느 Pod로?
```

### ConfigMap

```text
코드/이미지와 설정을 분리한다.

같은 이미지
   +
환경마다 다른 ConfigMap
   =
dev/prod 환경 분리
```

### env / envFrom

```text
env     → 하나씩 가져오기
envFrom → 통째로 가져오기
```

### 환경변수 변경

```text
ConfigMap 변경
      ↓
기존 Pod 환경변수 자동 변경 X
      ↓
Pod 재생성/재시작 필요
```

### 파일로 설정 전달

```text
ConfigMap
   ↓
volume
   ↓
volumeMount
   ↓
Container 내부 설정 파일
```

### ConfigMap / Secret

```text
일반 설정 → ConfigMap
비밀 정보 → Secret
```

---

# 오늘의 한 줄 정리

> **Ingress와 Service로 트래픽의 길을 만들고, ConfigMap과 Secret으로 애플리케이션의 설정을 이미지 밖으로 분리한다.**

특히 ConfigMap에서 가장 중요하게 기억할 것은:

> **"코드는 한 벌, 설정은 여러 벌."**

같은 이미지를 dev와 prod에서 그대로 사용하면서  
환경별 설정만 다르게 주는 것이 Kubernetes에서 ConfigMap을 사용하는 중요한 이유다.
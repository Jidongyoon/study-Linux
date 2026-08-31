# Docker & Kubernetes 인프라 종합 실습

## 🎯 오늘의 학습

오늘은 지금까지 배운 **Docker와 Kubernetes의 핵심 내용을 처음부터 다시 직접 실습**하면서 전체적인 인프라 흐름을 복습했다.

Docker에서는 컨테이너 실행부터 Port, Exec, Bind Mount, Volume, Network, Dockerfile, Docker Compose까지 실습했고, Kubernetes에서는 Pod, Deployment, Service, ConfigMap, Secret, Probe, Ingress를 직접 생성하고 연결했다.

오늘 실습의 전체적인 흐름은 다음과 같다.

```text
Docker
Image
  ↓
Container
  ├─ Port
  ├─ Bind Mount / Volume
  └─ Network
       ↓
Dockerfile
       ↓
Docker Compose

────────────────────────────

Kubernetes
Ingress
   ↓
Service
   ↓
Deployment
   ↓
Pod
   ↓
Container

ConfigMap / Secret
       ↓
      Pod

Readiness / Liveness Probe
       ↓
   Pod 상태 확인
```

---

# 🐳 1. Docker Container 기본 동작

먼저 `nginx:1.27` 이미지를 이용하여 컨테이너를 실행했다.

```bash
docker run -d --name total-web -p 8085:80 nginx:1.27
```

Port Mapping은 다음과 같은 구조이다.

```text
localhost:8085
      ↓
Host Port 8085
      ↓
Container Port 80
      ↓
nginx
```

`curl`을 이용하여 실제 nginx 페이지가 정상적으로 출력되는 것도 확인했다.

```bash
curl localhost:8085
```

이를 통해 Docker의 `-p` 옵션은

```text
호스트 포트 : 컨테이너 포트
```

형태로 연결한다는 것을 다시 확인했다.

---

# 🔄 2. Container 생명주기

실행 중인 Container를 중지하고 다시 시작하는 실습을 진행했다.

```bash
docker stop total-web

docker ps
docker ps -a

docker start total-web
```

Container를 `stop`한다고 Container 자체가 삭제되는 것은 아니다.

```text
Running
   ↓
docker stop
   ↓
Exited
   ↓
docker start
   ↓
Running
```

반면 다음과 같이 Container를 삭제하면 Container 자체가 사라진다.

```bash
docker rm -f total-web
```

하지만 Container를 삭제해도 원본 Image는 그대로 남아 있는 것을 `docker images`를 통해 확인했다.

```text
Image
  ↓
Container 생성
  ↓
Container 삭제

Image는 그대로 존재
```

이를 통해 **Image와 Container는 서로 다른 개념**이라는 것을 다시 확인했다.

---

# 🖥️ 3. docker exec와 Container 내부 수정

실행 중인 Container 내부에 접속했다.

```bash
docker exec -it total-web /bin/sh
```

nginx의 기본 HTML 파일을 수정했다.

```bash
echo "<h1>My Docker Lab</h1>" > /usr/share/nginx/html/index.html
```

이후 다시 `curl`을 실행했을 때 수정된 페이지가 출력되었다.

하지만 Container를 삭제하고 같은 Image로 다시 생성하자 수정했던 내용은 사라지고 nginx 기본 페이지가 다시 출력되었다.

```text
Image
  ↓
Container
  ↓
파일 수정
  ↓
Container 삭제
  ↓
수정 내용도 삭제
```

이를 통해 Container 내부의 writable layer에 저장된 데이터는 **Container가 삭제되면 함께 사라질 수 있다**는 것을 확인했다.

---

# 📂 4. Bind Mount

Container가 삭제되어도 데이터를 유지하는 방법 중 하나인 Bind Mount를 실습했다.

```bash
docker run -d \
  --name total-web \
  -p 8085:80 \
  -v "$(pwd)/index.html:/usr/share/nginx/html/index.html" \
  nginx:1.27
```

구조는 다음과 같다.

```text
Mac Host
index.html
    │
    │ Bind Mount
    ▼
Container
/usr/share/nginx/html/index.html
```

Host의 `index.html`을 수정하자 Container를 재시작하지 않아도 nginx 페이지가 바로 변경되는 것을 확인했다.

```bash
echo '<h1>Bind Mount Changed!</h1>' > index.html
```

이를 통해 Bind Mount는 **Host의 실제 파일 또는 디렉터리를 Container와 직접 연결하는 방식**이라는 것을 이해했다.

---

# 💾 5. Docker Volume

Docker가 직접 관리하는 저장 공간인 Volume도 실습했다.

Volume 생성:

```bash
docker volume create total-data
```

확인:

```bash
docker volume ls
docker volume inspect total-data
```

생성한 Volume을 nginx에 연결했다.

```bash
docker run -d \
  --name total-web \
  -p 8085:80 \
  -v total-data:/usr/share/nginx/html \
  nginx:1.27
```

Container 내부의 HTML 파일을 수정한 뒤 Container를 삭제했다.

이후 같은 Volume을 새로운 Container에 다시 연결했을 때 기존 HTML 내용이 그대로 유지되는 것을 확인했다.

```text
Container A
     │
     ▼
total-data Volume
     │
Container A 삭제
     │
     ▼
Volume은 유지
     │
     ▼
Container B
```

Bind Mount와 Volume의 차이는 다음과 같이 정리할 수 있다.

```text
Bind Mount
→ Host의 특정 경로를 직접 사용
→ 사용자가 Host 파일을 직접 관리

Volume
→ Docker가 저장 위치를 관리
→ Container와 독립적으로 데이터 유지
```

---

# 🌐 6. Docker Network

사용자 정의 Docker Network를 생성했다.

```bash
docker network create total-net
```

확인:

```bash
docker network ls
```

같은 Network에 Container를 생성했다.

```bash
docker run -d \
  --name net-a \
  --network total-net \
  ubuntu:24.04 sleep infinity

docker run -d \
  --name net-b \
  --network total-net \
  ubuntu:24.04 sleep infinity
```

`net-a` Container 내부에서 `net-b` 이름으로 통신했다.

```bash
ping -c 3 net-b
```

정상적으로 통신되는 것을 확인했다.

```text
total-net
   │
   ├── net-a
   │      │
   │      └── ping net-b
   │
   └── net-b
```

반면 같은 Network에 연결되지 않은 `net-c`는 처음에는 이름으로 통신할 수 없었다.

이후 실행 중인 `net-c`를 Network에 추가했다.

```bash
docker network connect total-net net-c
```

다시 `ping net-c`를 실행하자 정상적으로 통신되었다.

이를 통해 **같은 사용자 정의 Docker Network에 연결된 Container들은 Docker DNS를 통해 Container 이름으로 통신할 수 있다**는 것을 확인했다.

---

# 🏗️ 7. Dockerfile과 Image Build

기존 Dockerfile을 확인하면서 각 명령어의 역할을 복습했다.

```dockerfile
FROM python:3.11-slim

ENV HF_HOME=/cache \
    APP_ENV=prod

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

RUN useradd -r -u 10001 apisvc \
    && mkdir -p /cache \
    && chown -R apisvc /cache /app

USER apisvc

EXPOSE 8000

HEALTHCHECK --interval=10s --timeout=3s --start-period=60s \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/healthz')"

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

주요 명령어의 역할은 다음과 같다.

```text
FROM
→ 기반 Image 선택

WORKDIR
→ Container 내부 작업 디렉터리 설정

COPY
→ Host 파일을 Image 안으로 복사

RUN
→ docker build 과정에서 명령 실행

USER
→ Container를 실행할 사용자 지정

EXPOSE
→ Container가 사용하는 Port 정보 명시

HEALTHCHECK
→ Container 상태 검사

CMD
→ Container가 시작될 때 실행할 명령
```

특히 `RUN`과 `CMD`의 차이를 다시 확인했다.

```text
RUN
→ docker build 할 때 실행

CMD
→ docker run으로 Container가 시작될 때 실행
```

또한 `USER apisvc`를 이용하여 root 대신 필요한 최소 권한을 가진 사용자로 애플리케이션을 실행하는 구조도 확인했다.

---

# 📦 8. Docker Image Build와 Container 실행

Dockerfile을 이용하여 직접 Image를 생성했다.

```bash
docker build -t total-api:1 .
```

Image 확인:

```bash
docker images | grep total-api
```

생성한 Image로 Container를 실행했다.

```bash
docker run -d \
  --name total-api \
  -p 8000:8000 \
  total-api:1
```

API 상태 확인:

```bash
curl localhost:8000/healthz
```

정상적으로 다음과 같은 응답을 확인했다.

```json
{
  "status": "ok",
  "env": "prod"
}
```

전체 과정은 다음과 같다.

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
    ↓
Port Mapping
    ↓
Application
```

---

# 🐳 9. Docker Compose

Docker Compose를 이용하여 여러 Container를 하나의 YAML 파일로 관리하는 구조를 확인했다.

Compose에는 다음과 같은 설정들이 포함되어 있었다.

```text
API
├─ Image
├─ CPU / Memory 제한
├─ Restart Policy
├─ Secret
├─ env_file
├─ Volume
├─ Healthcheck
└─ Logging

Proxy
├─ nginx Image
├─ Port
├─ Bind Mount
├─ depends_on
└─ Logging
```

Compose 실행:

```bash
docker compose up -d
```

상태 확인:

```bash
docker compose ps
```

종료:

```bash
docker compose down
```

Docker Compose에서는 같은 Compose Network에 있는 Service들이 Service 이름을 이용하여 서로 통신할 수 있다는 것도 다시 확인했다.

```text
Client
   ↓
nginx proxy
   ↓
api:8000
   ↓
API Container
```

이를 통해 Docker Compose는 여러 Container의 실행 옵션을 매번 명령어로 작성하는 대신 **YAML을 이용하여 원하는 상태를 선언적으로 관리하는 방법**이라는 것을 이해했다.

---

# ☸️ 10. Kubernetes Namespace와 Pod

Kubernetes 실습에서는 먼저 Namespace를 확인했다.

```bash
kubectl get namespaces
```

`dev`, `prod`, `ingress-nginx` 등의 Namespace가 존재하는 것을 확인했다.

이후 `dev` Namespace에 nginx Pod를 직접 생성했다.

```bash
kubectl run total-nginx \
  --image=nginx:1.27 \
  -n dev
```

확인:

```bash
kubectl get pods -n dev
```

Pod를 직접 삭제했다.

```bash
kubectl delete pod total-nginx -n dev
```

다시 Pod 목록을 확인했을 때 삭제된 Pod가 자동으로 생성되지 않았다.

```text
직접 생성한 Pod
      ↓
Pod 삭제
      ↓
그대로 사라짐
```

이를 통해 실제 서비스에서는 Pod를 직접 관리하기보다는 Deployment와 같은 Controller를 이용하는 이유를 이해했다.

---

# 🔄 11. Kubernetes Deployment와 Self-Healing

Deployment YAML을 작성했다.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: total-deploy
  namespace: dev

spec:
  replicas: 2

  selector:
    matchLabels:
      app: total-nginx

  template:
    metadata:
      labels:
        app: total-nginx

    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

적용:

```bash
kubectl apply -f total-deploy.yaml
```

Deployment가 Pod 2개를 생성하는 것을 확인했다.

이후 Pod 하나를 직접 삭제했다.

```bash
kubectl delete pod <POD_NAME> -n dev
```

Pod가 하나 줄어들자 Deployment가 새로운 Pod를 자동으로 생성하여 다시 2개를 유지했다.

```text
Desired State
replicas: 2
     │
     ▼
Pod ① + Pod ②
     │
Pod ① 삭제
     │
     ▼
현재 상태 = 1개
원하는 상태 = 2개
     │
     ▼
새 Pod 생성
     │
     ▼
다시 2개 유지
```

이를 통해 Kubernetes의 핵심 개념인 **Desired State와 Self-Healing**을 직접 확인했다.

---

# 🌐 12. Kubernetes Service

Pod는 삭제되고 다시 생성될 수 있기 때문에 Pod의 이름이나 IP가 계속 바뀔 수 있다.

따라서 Pod에 안정적으로 접근하기 위해 Service를 생성했다.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: total-service
  namespace: dev

spec:
  selector:
    app: total-nginx

  ports:
    - port: 80
      targetPort: 80

  type: ClusterIP
```

Service에서 중요한 부분은 `selector`이다.

```text
Service
selector: app=total-nginx
          │
          ▼
Pod Label
app=total-nginx
```

Service의 Endpoint를 확인했다.

```bash
kubectl get endpoints total-service -n dev
```

두 Pod의 IP가 연결되어 있는 것을 확인했다.

```text
total-service
   │
   ├── 10.244.x.x:80
   └── 10.244.x.x:80
```

또한 임시 Pod를 실행하여 Service 이름으로 접근했다.

```bash
kubectl run test-client -n dev --rm -it \
  --image=busybox:1.36 \
  --restart=Never \
  -- wget -qO- http://total-service
```

nginx의 `Welcome to nginx!` 페이지가 출력되었다.

이를 통해 Service는 **Pod들을 찾아 안정적인 접근 지점을 제공하고 요청을 연결된 Pod 중 하나로 전달한다**는 것을 확인했다.

---

# ⚙️ 13. ConfigMap

애플리케이션의 일반 설정을 Image와 분리하기 위해 ConfigMap을 생성했다.

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: total-config
  namespace: dev

data:
  APP_ENV: "dev"
  LOG_LEVEL: "debug"
```

적용:

```bash
kubectl apply -f total-config.yaml
```

확인:

```bash
kubectl get cm total-config -n dev
kubectl describe cm total-config -n dev
```

ConfigMap의 값을 실제 Pod의 환경변수로 전달하기 위해 다음과 같이 `envFrom`을 사용했다.

```yaml
envFrom:
  - configMapRef:
      name: total-config
```

Pod 내부 확인:

```bash
kubectl exec config-test -n dev -- env | grep -E 'APP_ENV|LOG_LEVEL'
```

실제로 다음 값이 들어간 것을 확인했다.

```text
APP_ENV=dev
LOG_LEVEL=debug
```

이를 통해 **ConfigMap은 애플리케이션의 일반 설정을 Image와 분리하여 관리하는 리소스**라는 것을 이해했다.

---

# 🔐 14. Secret

API Token과 같은 민감한 정보를 관리하기 위해 Secret을 생성했다.

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: total-secret
  namespace: dev

type: Opaque

stringData:
  API_TOKEN: "my-secret-token"
```

Secret을 Pod에 연결할 때는 `secretKeyRef`를 사용했다.

```yaml
env:
  - name: API_TOKEN
    valueFrom:
      secretKeyRef:
        name: total-secret
        key: API_TOKEN
```

Pod 내부에서 실제 값을 확인했다.

```bash
kubectl exec secret-test -n dev -- env | grep API_TOKEN
```

결과:

```text
API_TOKEN=my-secret-token
```

ConfigMap과 Secret의 차이는 다음과 같이 정리할 수 있다.

```text
ConfigMap
→ 일반적인 설정값
→ APP_ENV, LOG_LEVEL 등

Secret
→ 민감한 설정값
→ Password, API Token 등
```

또한 Kubernetes Secret의 Base64 표현 자체가 암호화를 의미하는 것은 아니라는 점도 다시 확인했다.

---

# ❤️ 15. Readiness Probe와 Liveness Probe

Deployment에 Probe를 추가했다.

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

`kubectl describe pod`를 이용하여 실제 적용 상태를 확인했다.

```text
Liveness:  http-get http://:80/
Readiness: http-get http://:80/
```

두 Probe의 역할은 다음과 같다.

```text
Readiness Probe
→ 지금 요청을 받을 준비가 되었는가?
→ 실패하면 Service 요청 대상에서 제외

Liveness Probe
→ 애플리케이션이 정상적으로 살아 있는가?
→ 계속 실패하면 Container 재시작
```

Deployment YAML에 Probe를 추가하고 다시 적용했을 때 새로운 설정을 가진 Pod가 생성되었다.

처음에는 새로운 Pod가 `0/1 Running` 상태였다가 Probe가 성공한 뒤 `1/1 Running` 상태가 되는 것도 확인했다.

---

# 🔄 16. Deployment 변경과 Rolling Update

Deployment YAML을 수정하고 다시 적용했다.

```bash
kubectl apply -f total-deploy.yaml
```

기존 Pod를 직접 수정하는 것이 아니라 새로운 설정을 가진 Pod가 생성되고 정상적으로 준비된 뒤 기존 Pod가 제거되는 과정을 확인했다.

```text
기존 Pod 실행 중
      ↓
새로운 Pod 생성
      ↓
Readiness 확인
      ↓
새 Pod Ready
      ↓
기존 Pod 제거
```

이를 통해 Deployment가 변경될 때 **Rolling Update 방식으로 새로운 Pod로 교체할 수 있다**는 것을 확인했다.

---

# 🚪 17. Kubernetes Ingress

Service 앞에 Ingress를 추가하여 외부 요청을 Service로 전달하는 구조를 실습했다.

처음에는 다음과 같은 오류가 발생했다.

```text
host "_" and path "/" is already defined in ingress dev/demo
```

기존 `demo` Ingress가 이미 `/` 경로를 사용하고 있었기 때문에 새로운 Ingress와 경로가 충돌한 것이 원인이었다.

기존 Ingress를 확인했다.

```bash
kubectl get ingress -n dev
kubectl describe ingress demo -n dev
```

기존 설정은 `/`와 `/blue` 경로를 사용하고 있었다.

따라서 새로운 Ingress에서는 `/total` 경로를 사용하도록 수정했다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: total-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  ingressClassName: nginx

  rules:
    - http:
        paths:
          - path: /total
            pathType: Prefix
            backend:
              service:
                name: total-service
                port:
                  number: 80
```

Ingress의 핵심 구조는 다음과 같다.

```text
Ingress
   │
   │ /total
   ▼
total-service:80
   │
   ▼
nginx Pod
```

---

# 🛠️ 18. Ingress 접속 장애 확인

Ingress가 생성된 뒤 Minikube IP를 확인했다.

```bash
minikube ip
```

```text
192.168.49.2
```

하지만 Mac에서 다음 주소로 직접 접근했을 때 연결되지 않았다.

```bash
curl http://192.168.49.2/total
```

Ingress Controller의 상태를 확인했다.

```bash
kubectl get pods -n ingress-nginx
```

Ingress Controller는 정상적으로 `Running` 상태였다.

따라서 Ingress 설정 자체의 문제인지 확인하기 위해 Port Forward를 이용했다.

```bash
kubectl port-forward \
  -n ingress-nginx \
  svc/ingress-nginx-controller \
  8088:80
```

다른 터미널에서 다음 요청을 실행했다.

```bash
curl http://localhost:8088/total
```

최종적으로 nginx의

```text
Welcome to nginx!
```

페이지가 정상적으로 출력되었다.

이를 통해 실제 요청이 다음 경로로 정상적으로 전달되고 있음을 확인했다.

```text
Mac
 │
 │ localhost:8088/total
 ▼
Port Forward
 │
 ▼
Ingress Controller
 │
 ▼
Ingress
 │
 │ /total
 ▼
total-service
 │
 ├─────────────┐
 ▼             ▼
Pod ①         Pod ②
nginx         nginx
```

---

# 🐛 19. 오늘 발생한 주요 오류와 해결

오늘 실습에서는 정상적인 실행뿐만 아니라 여러 오류를 직접 경험하면서 원인을 확인했다.

### Docker Container를 잘못 종료

Container ID만 보고 `docker stop`을 실행하다가 nginx가 아닌 Minikube Container를 중지했다.

이를 통해 Container를 조작하기 전에는 다음 명령으로 이름과 Image를 먼저 확인하는 습관이 중요하다는 것을 배웠다.

```bash
docker ps
```

가능하면 ID보다 Container 이름을 사용하는 것이 실수를 줄이는 데 도움이 된다.

### Shell Redirection 오류

Container 내부 파일을 수정할 때 `>`가 Host Shell에서 먼저 처리되는 문제를 경험했다.

Container 내부에 접속한 뒤 정확한 경로를 지정하여 해결했다.

```bash
docker exec -it total-web /bin/sh
```

### Docker Image Tag 오타

```text
ubuntu:24:04
```

처럼 작성하여 Image reference 오류가 발생했다.

정확한 Tag는 다음과 같았다.

```text
ubuntu:24.04
```

### YAML 들여쓰기 오류

Service YAML에서 `ports`와 `type`이 `selector` 아래에 들어가면서 오류가 발생했다.

Kubernetes YAML에서는 다음과 같이 계층 구조가 정확해야 한다.

```text
spec
├── selector
├── ports
└── type
```

또한 Service에서는 `port`가 아니라 `ports` 필드를 사용한다는 것도 확인했다.

### kind 대소문자 오류

Pod YAML에서 다음과 같이 작성하여 오류가 발생했다.

```yaml
kind: pod
```

정확한 값은 다음과 같다.

```yaml
kind: Pod
```

Kubernetes YAML의 값은 대소문자를 정확하게 작성해야 한다.

### kubectl exec의 `--`

다음 명령은 잘못된 명령이었다.

```bash
kubectl exec secret-test -n dev --env
```

정확한 형태는 다음과 같다.

```bash
kubectl exec secret-test -n dev -- env
```

`--` 뒤에 오는 부분은 Container 내부에서 실행할 명령이라는 것을 이해했다.

---

# 🔗 20. Docker와 Kubernetes 연결해서 이해하기

오늘 Docker와 Kubernetes를 연속해서 실습하면서 두 기술의 관계도 조금 더 명확하게 이해할 수 있었다.

Docker에서는 직접 Container를 실행하고 관리한다.

```text
Dockerfile
   ↓
Image
   ↓
Container
```

Kubernetes에서는 이러한 Container를 Pod 안에서 실행하고 여러 Kubernetes 리소스가 관리한다.

```text
Ingress
   ↓
Service
   ↓
Deployment
   ↓
Pod
   ↓
Container
   ↓
Image
```

Docker에서 배웠던 Image, Container, Network, Volume 등의 개념이 Kubernetes를 이해하는 기반이 된다는 것을 확인했다.

특히 Kubernetes는 단순히 Container를 실행하는 도구가 아니라 **Container 기반 애플리케이션이 원하는 상태를 계속 유지하도록 관리하는 시스템**이라는 점이 중요했다.

---

# 💡 오늘의 배운 점

오늘은 Docker와 Kubernetes의 개별 명령어를 외우는 것보다 **각 기술이 왜 필요한지와 서로 어떻게 연결되는지**를 실습을 통해 확인했다.

Docker에서는

```text
Image
→ Container
→ Port
→ Storage
→ Network
→ Dockerfile
→ Docker Compose
```

의 흐름을 복습했다.

Kubernetes에서는

```text
Ingress
   ↓
Service
   ↓
Deployment
   ↓
Pod
   ↓
Container
```

구조를 직접 만들었다.

또한

```text
ConfigMap / Secret
        ↓
       Pod

Readiness / Liveness Probe
        ↓
     Pod 상태 관리
```

까지 연결하면서 Kubernetes의 주요 리소스가 하나의 애플리케이션을 운영하기 위해 어떻게 협력하는지 이해할 수 있었다.

실습 과정에서 오류가 발생했을 때 단순히 명령어를 다시 실행하는 것이 아니라 **어느 구간에서 문제가 발생했는지 확인하는 과정도 중요하다**는 것을 배웠다.

예를 들어 Ingress 접속 문제에서는

```text
Client
 ↓
Ingress Controller
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

순서로 구간을 나누어 확인하면서 문제의 위치를 좁힐 수 있었다.

---

# 📌 오늘의 핵심 정리

**Docker는 Image를 기반으로 Container를 만들고 실행하는 환경을 제공하며, Kubernetes는 이러한 Container를 Pod 단위로 실행하고 Deployment, Service, Ingress 등의 리소스를 이용해 원하는 상태로 지속적으로 관리한다.**

특히 오늘 실습을 통해 다음 흐름을 직접 확인했다.

```text
사용자 요청
     ↓
Ingress
     ↓
Service
     ↓
Deployment가 관리하는 Pod
     ↓
Container
     ↓
Application
```

그리고 애플리케이션 운영에 필요한 설정과 상태 관리는

```text
ConfigMap → 일반 설정
Secret    → 민감 정보
Probe     → 상태 확인
```

으로 분리할 수 있다는 것을 배웠다.

오늘 실습을 통해 **Docker에서 Container 하나를 실행하는 단계에서 시작하여 Kubernetes에서 여러 Pod를 관리하고 외부 요청을 애플리케이션까지 전달하는 전체적인 인프라 흐름을 한 번에 연결해서 이해할 수 있었다.**
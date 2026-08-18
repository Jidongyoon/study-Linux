# 🐳 Docker 학습 기록 - Volume / Compose / Nginx / Healthcheck

## 📌 오늘 학습한 내용

오늘은 Docker의 **Volume, Docker Compose, Nginx Reverse Proxy, Healthcheck**를 연결하여 하나의 서비스 구조를 구성해보았다.

단순히 컨테이너를 실행하는 것에서 끝나는 것이 아니라,
여러 컨테이너를 연결하고 데이터를 유지하며 서비스 상태에 따라 실행 순서를 관리하는 방법을 실습했다.

---

## 💾 1. Docker Volume

컨테이너 내부의 데이터는 컨테이너가 삭제되면 함께 사라질 수 있다.

Volume을 사용하면 컨테이너와 데이터를 분리하여 관리할 수 있다.

이번 실습에서는 `hf-cache` Volume을 컨테이너의 `/cache`에 연결했다.

```yaml
volumes:
  - hf-cache:/cache
```

실제로 컨테이너를 삭제한 후 다시 생성해도 `/cache`의 데이터가 그대로 유지되는 것을 확인했다.

> **Container는 다시 생성할 수 있지만 중요한 데이터는 Volume에 보존할 수 있다.**

---

## 🐳 2. Docker Compose

Docker Compose를 이용하여 **FastAPI와 Nginx 컨테이너를 함께 관리**했다.

```bash
docker compose up -d
```

Compose를 실행하면 필요한 컨테이너와 Network를 한 번에 생성할 수 있다.

또한 같은 Compose Network에 있는 컨테이너들은 IP 주소 대신 **서비스 이름**으로 통신할 수 있다.

```text
nginx → api:8000
```

---

## 🌐 3. Nginx Reverse Proxy

FastAPI의 `8000` 포트를 외부에 직접 공개하지 않고,
Nginx의 `8080` 포트만 외부에 공개하도록 구성했다.

```text
Client
   ↓
localhost:8080
   ↓
Nginx
   ↓
Docker Network
   ↓
api:8000
   ↓
FastAPI
```

Nginx에서는 다음과 같이 FastAPI로 요청을 전달했다.

```nginx
proxy_pass http://api:8000;
```

테스트 결과:

```bash
curl localhost:8080/healthz
```

→ 정상 응답 ✅

```bash
curl localhost:8000/healthz
```

→ 직접 접근 실패 ✅

이를 통해 **외부에는 Nginx만 공개하고 API는 내부 Network에서 통신하도록 구성할 수 있다는 것**을 확인했다.

---

## ❤️ 4. Healthcheck + depends_on

컨테이너가 실행되었다고 해서 애플리케이션이 바로 요청을 처리할 수 있는 것은 아니다.

따라서 API의 Healthcheck 결과를 이용해 Nginx의 실행 순서를 설정했다.

```yaml
depends_on:
  api:
    condition: service_healthy
```

실행 흐름은 다음과 같다.

```text
FastAPI 시작
     ↓
Healthcheck
     ↓
Healthy
     ↓
Nginx 시작
```

실제로 Compose 실행 시 다음 순서를 확인했다.

```text
✔ Container myapi-api-1    Healthy
✔ Container myapi-proxy-1  Started
```

> **컨테이너가 실행 중인 것과 서비스가 정상적으로 준비된 것은 다르다.**

---

## 🧠 오늘 느낀 점

Volume, Compose, Network, Nginx, Healthcheck를 직접 연결해보면서
Docker의 각 기능들이 실제 서비스에서는 서로 연결되어 사용된다는 것을 이해할 수 있었다.

특히 단순히 명령어를 외우기보다 **각 설정이 왜 필요한지 이해하는 것이 중요하다고 느꼈다.**

---

## 🎯 오늘의 한 줄

> **Docker Compose를 이용하면 여러 컨테이너의 Network, Volume, 상태와 실행 순서를 하나의 서비스 구성으로 관리할 수 있다.**
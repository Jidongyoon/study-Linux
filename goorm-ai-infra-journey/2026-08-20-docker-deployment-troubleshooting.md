# 🐳 Docker 서비스 배포 및 트러블슈팅 실습

## 📌 학습 주제

**Docker 이미지 빌드부터 서버 배포, Nginx Reverse Proxy와 Docker Compose를 활용한 서비스 구성**

오늘 수업에서는 Dockerfile을 이용해 애플리케이션 이미지를 만들고, 생성한 이미지를 실제 서버에서 컨테이너로 실행하는 과정을 실습했다.

단순히 컨테이너를 실행하는 것에서 끝나는 것이 아니라 **Docker Network, Nginx Reverse Proxy, Docker Compose를 이용해 여러 컨테이너가 하나의 서비스로 동작하는 전체적인 흐름**을 학습했다.

---

## 1. Dockerfile을 이용한 이미지 생성

애플리케이션을 컨테이너에서 실행하기 위해 `Dockerfile`을 작성하고 직접 이미지를 빌드했다.

Dockerfile에는 Base Image, 패키지 설치, 애플리케이션 파일 복사, 환경변수, Health Check, 실행 명령어 등을 설정할 수 있다.

    Application
        ↓
    Dockerfile
        ↓
    docker build
        ↓
    Docker Image

이미지 빌드:

    docker build -t myapi:1 .

생성된 이미지 확인:

    docker images

이를 통해 Docker 이미지는 단순히 프로그램 파일을 저장하는 것이 아니라 **애플리케이션을 실행하는 데 필요한 환경과 실행 방법까지 포함하는 배포 단위**라는 것을 이해했다.

---

## 2. 컨테이너 실행과 포트 매핑

생성한 이미지를 이용해 실제 서버에서 컨테이너를 실행했다.

    docker run -d --name myapi -p 8000:8000 myapi:1

구조는 다음과 같다.

    Client
       ↓
    Host :8000
       ↓
    Container :8000
       ↓
    FastAPI

컨테이너 상태는 다음 명령어로 확인했다.

    docker ps
    docker ps -a

그리고 `curl`을 이용해 실제 애플리케이션이 정상적으로 응답하는지도 확인했다.

    curl localhost:8000/healthz

정상 응답:

    {"status":"ok"}

여기서 **컨테이너가 실행 중인 것과 내부 애플리케이션이 정상적으로 동작하는 것은 별도로 확인해야 한다**는 점을 알게 되었다.

---

## 3. Health Check

Dockerfile에 `HEALTHCHECK`를 설정하여 컨테이너 내부 애플리케이션이 실제로 정상 동작하는지 확인했다.

상태 확인:

    docker ps

정상적으로 동작하면 다음과 같이 확인할 수 있다.

    Up (healthy)

Health Check를 통해 단순히 컨테이너 프로세스가 실행 중인지 확인하는 것을 넘어 **내부 서비스가 실제 요청을 처리할 수 있는 상태인지 확인할 수 있다.**

---

## 4. Nginx Reverse Proxy

API 컨테이너 앞에 Nginx를 배치하여 Reverse Proxy 구조를 구성했다.

    Client
       ↓
    Nginx
    Reverse Proxy
       ↓
    FastAPI Container

Nginx는 외부 요청을 먼저 받은 뒤 내부의 API 서비스로 요청을 전달한다.

이를 통해 Nginx가 단순한 웹 서버뿐만 아니라 **외부 요청과 내부 애플리케이션을 연결하는 Reverse Proxy 역할**도 할 수 있다는 것을 이해했다.

---

## 5. Docker Network와 내부 DNS

Docker Compose로 구성된 컨테이너들은 같은 Docker Network 안에서 서로 통신할 수 있다.

이때 컨테이너의 IP 주소를 직접 지정하는 대신 **Compose에 정의된 서비스 이름을 이용해 통신할 수 있다.**

예를 들어 API 서비스의 이름이 `api`라면 Nginx에서는 다음과 같이 접근할 수 있다.

    api:8000

통신 과정:

    Nginx
       ↓
    api:8000
       ↓
    Docker Internal DNS
       ↓
    API Container

컨테이너는 다시 생성될 때 IP 주소가 변경될 수 있기 때문에 **고정 IP를 사용하는 것보다 서비스 이름과 Docker 내부 DNS를 이용하는 것이 중요하다**는 점을 배웠다.

---

## 6. Docker Compose를 이용한 서비스 구성

API와 Nginx를 각각 따로 실행하는 대신 Docker Compose를 이용해 여러 컨테이너를 하나의 서비스 구성으로 관리했다.

    Docker Compose
        │
        ├── API
        │    └── FastAPI
        │
        └── Proxy
             └── Nginx

`compose.yaml`에 각 서비스의 이미지, 포트, 볼륨, 의존 관계 등을 정의하고 다음 명령으로 실행했다.

    docker compose up -d

상태 확인:

    docker compose ps

로그 확인:

    docker compose logs

이를 통해 Docker Compose는 단순히 여러 이미지를 묶는 것이 아니라 **여러 컨테이너의 실행 환경과 연결 관계를 하나의 설정 파일로 관리하는 도구**라는 것을 이해했다.

---

## 7. 포트 매핑과 포트 충돌

실습을 통해 Host Port와 Container Port의 차이도 확인했다.

예를 들어 다음과 같이 실행할 수 있다.

    docker run -d --name myapi-port -p 8001:9999 myapi:2

이 경우 Docker 컨테이너 자체는 실행될 수 있지만 애플리케이션이 실제로 `8000` 포트에서 동작한다면 `9999` 포트에는 서비스가 존재하지 않는다.

    Host :8001
        ↓
    Container :9999
        ↓
    서비스 없음
        ↓
    접속 실패

또한 이미 사용 중인 Host Port를 다른 컨테이너가 사용하려 하면 다음과 같은 오류가 발생했다.

    port is already allocated

사용 중인 포트는 다음 명령으로 확인할 수 있다.

    ss -tlnp

특정 포트 확인:

    ss -tlnp | grep 8080

이를 통해 **하나의 Host Port를 동시에 여러 컨테이너가 사용할 수 없다는 것**을 확인했다.

---

## 8. Nginx Upstream 장애

Nginx 설정에서 잘못된 서비스 이름을 지정했을 때 다음과 같은 오류가 발생했다.

    host not found in upstream "myapi:8000"

Docker Compose에서는 API 서비스 이름이 `api`였지만 Nginx 설정에서 `myapi`를 찾도록 설정되어 있었기 때문에 Docker 내부 DNS가 해당 서비스를 찾을 수 없었다.

올바른 구조:

    Nginx
       ↓
    api:8000
       ↓
    Docker DNS
       ↓
    API Container

이를 통해 `host not found in upstream` 오류가 발생하면 **Nginx 설정의 서비스 이름과 Docker Network/DNS를 확인해야 한다**는 것을 배웠다.

---

## 9. 실습 중 경험한 장애

오늘 실습에서는 여러 오류를 직접 확인했다.

- Dockerfile 문법 오류 → 이미지 빌드 실패
- `COPY` 대상 파일 누락 → Build Context 확인
- `requirements.txt` 내용 오류 → 패키지 설치 실패
- 잘못된 `USER` 설정 → 컨테이너 시작 실패
- 잘못된 포트 매핑 → 컨테이너는 실행되지만 서비스 접근 실패
- `port is already allocated` → Host Port 충돌
- `host not found in upstream` → Nginx 서비스 이름/Docker DNS 문제

이러한 장애를 직접 확인하면서 에러 메시지를 읽고 문제가 발생한 위치를 좁혀가는 연습을 할 수 있었다.

---

## 10. Docker 트러블슈팅 기본 흐름

오늘 실습을 통해 서비스 장애가 발생했을 때 다음과 같은 순서로 확인할 수 있다는 것을 정리했다.

    장애 발생
        ↓
    언제부터 발생했는가?
        ↓
    최근 변경사항이 있는가?
        ↓
    서비스 / Container가 살아있는가?
        ↓
    CPU / Memory / Disk는 정상인가?
        ↓
    로그에 어떤 Error가 있는가?
        ↓
    Port는 정상적으로 LISTEN 중인가?
        ↓
    Docker Network / DNS는 정상인가?
        ↓
    외부 서비스 연결은 정상인가?

주요 확인 명령어:

    docker ps -a
    docker logs <container>
    docker compose ps
    docker compose logs
    docker stats
    ss -tlnp
    free -h
    df -h
    curl -v <URL>
    docker network ls
    docker inspect <container>

장애가 발생했다고 바로 설정을 수정하기보다 **현재 상태와 로그를 먼저 확인하고 문제의 범위를 단계적으로 좁혀가는 것이 중요하다.**

---

## 11. 서비스 업데이트와 무중단 배포

새로운 버전의 이미지를 배포할 때 기존 컨테이너를 종료하고 새로운 컨테이너를 실행하면 새로운 애플리케이션이 준비되는 동안 서비스가 일시적으로 중단될 수 있다.

    기존 Container
         ↓
    Container 종료
         ↓
    새로운 Container 생성
         ↓
    Application 시작
         ↓
    Health Check
         ↓
    서비스 정상화

이를 통해 실제 운영 환경에서는 단순히 새로운 이미지를 배포하는 것뿐만 아니라 **서비스 중단을 최소화하기 위한 무중단 배포 방식도 고려해야 한다**는 점을 알게 되었다.

Rolling Update, Blue-Green Deployment, Load Balancing 등의 방식이 이러한 문제를 해결하기 위해 사용될 수 있다.

---

## 📝 오늘의 핵심 정리

오늘 실습에서는 Docker의 개별 명령어를 외우는 것보다 **애플리케이션이 실제 서비스로 배포되는 전체적인 흐름을 이해하는 것**이 중요하다는 것을 배웠다.

    Application
         ↓
    Dockerfile
         ↓
    Docker Image
         ↓
    Container
         ↓
    Docker Network
         ↓
    Nginx Reverse Proxy
         ↓
    External Client

또한 여러 장애를 직접 경험하면서 다음과 같은 트러블슈팅 흐름을 연습했다.

    상태 확인
       ↓
    로그 확인
       ↓
    서버 자원 확인
       ↓
    포트 확인
       ↓
    네트워크 / DNS 확인
       ↓
    원인 특정

Dockerfile이나 Compose의 모든 문법을 암기하기보다는 **각 설정이 왜 필요한지, 컨테이너들이 어떤 관계로 동작하는지, 장애가 발생했을 때 어디부터 확인해야 하는지를 이해하는 것이 중요하다**는 점을 오늘 실습을 통해 배울 수 있었다.
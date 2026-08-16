# 🐳 Docker 핵심 실습

> 2026-08-16 학습 기록

오늘은 Docker의 포트 매핑, 네트워크, 컨테이너 생명주기, Dockerfile, 이미지와 컨테이너의 차이, Docker Hub, VM과 컨테이너 구조를 실습했다.

---

## 1. Port Mapping

nginx 컨테이너를 실행했다.

```
docker run -d --name mission-nginx nginx
```

포트 매핑이 없을 때는 컨테이너 내부의 `80/tcp`만 존재해서 Host에서 접근할 수 없었다.

```
docker run -d \
  --name mission-nginx \
  -p 8080:80 \
  nginx
```

결과:

```
0.0.0.0:8080->80/tcp
```

즉,

```
Host 8080
   ↓
Container 80
   ↓
  nginx
```

`http://localhost:8080`으로 정상 접속되는 것을 확인했다.

---

## 2. 127.0.0.1과 0.0.0.0

컨테이너 내부 서버를 다음과 같이 실행했다.

```
python -m http.server 8000 --bind 127.0.0.1
```

Port Mapping이 있어도 외부 접속이 되지 않았다.

이후:

```
python -m http.server 8000 --bind 0.0.0.0
```

으로 변경하자 정상 접속되었다.

핵심:

> `127.0.0.1`은 컨테이너 자기 자신이고, 외부 요청을 받으려면 서비스가 `0.0.0.0` 등에 바인딩되어 있어야 한다.

---

## 3. Docker Network

기본 bridge 네트워크에서는 컨테이너 IP를 이용해 통신했다.

```
docker exec net-a ping -c 3 172.17.0.4
```

사용자 정의 네트워크를 생성했다.

```
docker network create mission-net
```

컨테이너를 연결했다.

```
docker run -dit \
  --name user-a \
  --network mission-net \
  ubuntu:24.04 bash

docker run -dit \
  --name user-b \
  --network mission-net \
  ubuntu:24.04 bash
```

이후 이름으로 통신했다.

```
docker exec user-a ping -c 3 user-b
```

Docker DNS가 `user-b`를 실제 IP로 변환해 정상 통신했다.

핵심:

> 컨테이너 IP는 변경될 수 있으므로 현업에서는 IP보다 서비스 이름을 사용하는 것이 관리하기 쉽다.

---

## 4. 컨테이너 생명주기

컨테이너 실행:

```
docker run -dit \
  --name lifecycle-test \
  ubuntu:24.04 bash
```

정지:

```
docker stop lifecycle-test
```

전체 상태 확인:

```
docker ps -a
```

다시 실행:

```
docker start lifecycle-test
```

정리하면:

```
docker run
→ 생성 + 실행

docker stop
→ 정지

docker start
→ 기존 컨테이너 재실행

docker rm
→ 삭제
```

`stop`과 `rm`은 서로 다른 작업이다.

---

## 5. PID 1과 docker exec

컨테이너 내부 프로세스를 확인했다.

```
docker exec lifecycle-test ps -ef
```

결과:

```
PID 1 = bash
```

컨테이너의 PID 1은 메인 프로세스이며 이 프로세스의 생명주기와 컨테이너의 실행 상태가 연결된다.

`docker exec`는 실행 중인 컨테이너에서 추가 명령을 실행한다.

```
docker exec lifecycle-test hostname
```

반면:

```
docker run ubuntu:24.04 hostname
```

은 새로운 컨테이너를 생성하고 `hostname`을 메인 프로세스로 실행한다.

핵심:

> `run`은 새 컨테이너 생성, `exec`는 기존 컨테이너 안에서 추가 명령 실행이다.

---

## 6. Dockerfile

기존에 SSH로 서버에 접속해 직접 하던 작업을 Dockerfile로 자동화했다.

| 지시어       | 기존 서버 작업      |
| --------- | ------------- |
| `FROM`    | 실행 환경 준비      |
| `WORKDIR` | 작업 디렉터리 생성/이동 |
| `COPY`    | 애플리케이션 배포     |
| `RUN`     | 패키지 설치, 계정 생성 |
| `ENV`     | 환경변수 설정       |
| `EXPOSE`  | 서비스 Port 명시   |
| `USER`    | 서비스 실행 계정 지정  |
| `CMD`     | 애플리케이션 실행     |

이미지를 빌드했다.

```
docker build -t ai-service:0.1 .
```

실행:

```
docker run -d \
  --name ai-api \
  -p 8000:8000 \
  ai-service:0.1
```

API와 Health Check가 정상적으로 응답하는 것을 확인했다.

---

## 7. root 대신 일반 사용자 사용

Dockerfile에서 서비스 전용 사용자를 생성했다.

```
RUN useradd --create-home --uid 10001 appuser

USER appuser
```

확인:

```
docker exec ai-api whoami
```

결과:

```
appuser
```

애플리케이션이 root 권한을 꼭 필요로 하지 않는다면 일반 사용자로 실행하는 것이 안전하다.

> 최소 권한 원칙을 적용해 보안 사고가 발생했을 때 피해 범위를 줄인다.

---

## 8. Image Layer 확인

이미지 생성 과정을 확인했다.

```
docker history ai-service:0.1
```

`COPY`, `RUN`, `ENV`, `USER`, `CMD` 등의 단계가 이미지 생성 과정에 반영되어 있었다.

AI 라이브러리를 설치한 `RUN` Layer의 크기가 특히 큰 것을 확인했다.

핵심:

> Docker 이미지는 여러 Layer가 쌓여 만들어지고, 무거운 의존성은 이미지 크기와 배포 시간에 영향을 준다.

---

## 9. Image와 Container

같은 이미지로 두 컨테이너를 실행했다.

```
ai-service:0.1
     │
  ┌──┴──┐
  ↓     ↓
ai-one ai-two
```

`ai-one`에 파일을 만들었다.

```
docker exec ai-one \
  sh -c 'echo "hello" > /tmp/test.txt'
```

`ai-two`에서는 해당 파일이 보이지 않았다.

각 컨테이너는 같은 이미지를 사용하더라도 독립적인 writable layer를 가진다.

핵심:

> Image는 공통 Template이고 Container는 Image를 기반으로 실행된 독립적인 Instance이다.

---

## 10. Tag와 Docker Hub

같은 이미지에 여러 Tag를 붙였다.

```
ai-service:0.1
ai-service:0.2
ai-service:latest
```

Tag가 달라도 같은 IMAGE ID를 가리킬 수 있다.

`latest` 역시 자동으로 최신 버전을 찾아주는 기능이 아니라 단순한 Tag이다.

따라서 운영에서는:

```
ai-service:1.0.0
```

처럼 명확한 버전을 사용하는 것이 안전하다.

Docker Hub 이미지도 직접 받아 실행했다.

```
docker pull hello-world:latest
docker run --rm hello-world:latest
```

직접 만든 이미지에도 Repository Tag를 붙여 Push를 진행했다.

```
docker tag \
  ai-service:0.1 \
  soulbabyg/ai-service:0.1

docker push soulbabyg/ai-service:0.1
```

AI 의존성 때문에 이미지 Layer가 매우 커져 Push 시간이 크게 증가하는 것도 확인했다.

---

## 11. VM과 Container

VM은 각각 별도의 Guest OS와 Kernel을 가진다.

Container는 Host의 Linux Kernel을 공유한다.

| VM          | Container      |
| ----------- | -------------- |
| Guest OS 존재 | Host Kernel 공유 |
| OS 부팅 필요    | 프로세스 실행 중심     |
| 상대적으로 무거움   | 상대적으로 가벼움      |
| VM 단위 격리    | 프로세스 단위 격리     |

따라서 컨테이너는 작은 VM이라기보다 **격리된 프로세스 실행 환경**에 가깝다.

---

## 12. Mac에서 Docker가 실행되는 구조

실제 환경을 확인했다.

```
docker version
```

Client:

```
OS/Arch: darwin/arm64
```

Server:

```
OS/Arch: linux/arm64
```

`docker info`에서는:

```
Kernel Version: 6.12.76-linuxkit
OSType: linux
Name: docker-desktop
```

을 확인했다.

현재 구조는 다음과 같다.

```
macOS
  ↓
Docker Desktop
  ↓
Linux VM
  ↓
Docker Engine
  ↓
Linux Container
```

macOS에는 Linux Kernel이 없기 때문에 Docker Desktop의 Linux 환경에서 컨테이너가 실행된다.

---

# 🎯 오늘의 핵심

오늘 Docker의 전체 흐름을 다음과 같이 정리할 수 있었다.

```
Dockerfile
    ↓
docker build
    ↓
  Image
    ↓
docker run
    ↓
Container
    │
    ├── PID 1
    ├── Network
    ├── Port Mapping
    └── Writable Layer
```

가장 중요하게 이해한 것은:

> **Docker Image는 실행 환경을 재현하기 위한 Template이고, Container는 그 이미지를 기반으로 실행되는 격리된 프로세스 환경이다.**

또한 빈 서버에서 사람이 직접 하던 설정을 Dockerfile로 코드화하면 동일한 실행 환경을 반복해서 만들고 배포할 수 있다는 점을 이해했다.

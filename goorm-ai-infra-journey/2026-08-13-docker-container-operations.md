# 🐳 Docker 컨테이너 · 설정 · 데이터 · 상태 관리

> 2026-08-13 학습 기록

## 1. Docker 이미지와 컨테이너

오늘은 Docker Image를 기반으로 실제 컨테이너를 실행하고 관리하는 방법을 학습했다.

같은 `myapi:0.3` 이미지를 사용하면서 실행 시 환경변수를 다르게 주입하여 개발용과 운영용 컨테이너를 각각 실행했다.

    myapi:0.3
       ├── myapi-dev  → APP_ENV=dev
       └── myapi      → APP_ENV=prod

이미지는 애플리케이션의 기본 틀이고 컨테이너는 그 이미지를 실행한 실제 환경이라는 것을 이해했다.

---

## 2. 환경변수와 설정 관리

`-e` 옵션을 이용해 컨테이너 실행 시 환경변수를 주입했다.

    -e APP_ENV=dev
    -e APP_NAME=myapi-dev

또한 `--env-file`을 사용해 여러 환경변수를 파일로 관리하는 방법을 실습했다.

    --env-file ~/docker-config/dev.env

환경에 따라 달라지는 설정을 이미지와 분리하면 하나의 이미지를 개발과 운영 환경에서 재사용할 수 있다.

---

## 3. Secret과 이미지 설정

Dockerfile에 API Key를 직접 넣는 경우를 실습했다.

    ENV API_KEY=super-secret-123

이후 `docker inspect`를 이용해 이미지 설정에 해당 값이 남아 있는 것을 확인했다.

    docker inspect secret-test:0.1 --format '{{json .Config.Env}}'

이를 통해 비밀번호나 API Key 같은 민감정보를 이미지에 포함하면 이미지가 공유되거나 배포될 때 정보가 노출될 수 있다는 것을 이해했다.

---

## 4. Volume과 Bind Mount

컨테이너가 삭제되어도 데이터를 유지할 수 있도록 Volume을 사용했다.

    -v volume-test:/data

또한 호스트의 디렉터리를 컨테이너와 연결하는 Bind Mount도 실습했다.

    -v ~/docker-bind-test:/data

### 저장 방식

| 구분 | Volume | Bind Mount |
|---|---|---|
| 비유 | Docker가 관리하는 창고 | 내 컴퓨터의 작업 책상 |
| 용도 | 데이터 보존 | 소스 코드·파일 공유 |
| 개발 환경 | 보통 | 편리함 |

Volume은 데이터 보존에 적합하고 Bind Mount는 개발 중인 파일을 컨테이너와 공유할 때 편리하다.

---

## 5. 파일 소유자와 권한

호스트에서 생성한 파일을 Bind Mount로 컨테이너에서 확인하고 UID/GID와 파일 권한을 비교했다.

Linux에서는 사용자 이름보다 UID/GID를 기준으로 사용자를 구분하고 `rwx` 권한을 기준으로 파일 접근을 판단한다.

UID가 다르고 필요한 권한이 없다면 `Permission denied`가 발생할 수 있으며, 이는 교과목 01에서 배운 Linux의 소유자·그룹·기타 사용자 권한과 같은 원리이다.

---

## 6. Healthcheck

컨테이너의 프로세스가 실행 중인 것과 실제 서비스가 정상적으로 응답하는 것은 다르다는 것을 확인했다.

    docker inspect myapi-dev --format '{{.State.Health.Status}}'

컨테이너가 처음에는 `starting` 상태였다가 정상적인 Healthcheck를 통과하면 `healthy`로 변경되는 것을 확인했다.

즉,

    프로세스 실행
    → Healthcheck 확인
    → 정상 응답
    → healthy

순서로 서비스 상태를 판단할 수 있다.

---

## 7. CPU와 메모리 제한

컨테이너 실행 시 CPU와 메모리 사용량에 제한을 설정했다.

    --memory=128m
    --cpus=0.5

이후 `docker stats`를 사용해 실제 자원 사용량을 관찰했다.

    docker stats resource-test

`CPU %`와 `MEM USAGE / LIMIT`을 통해 현재 컨테이너가 사용하는 자원과 설정된 제한을 확인했다.

메모리 제한을 초과하면 OOM Kill이 발생하여 프로세스가 강제 종료될 수 있으며, 이는 Ubuntu에서 관리자가 `kill` 명령으로 직접 프로세스를 종료하는 것과 달리 시스템이 메모리 자원을 보호하기 위해 자동으로 종료하는 방식이다.

---

## 🎯 오늘의 핵심

오늘은 Docker를 단순히 프로그램을 실행하는 도구로 보는 것이 아니라,

    이미지
    → 컨테이너
    → 설정
    → 데이터
    → 권한
    → Healthcheck
    → 자원 제한

까지 함께 관리하는 **서비스 운영 환경**으로 이해하는 데 집중했다.

> **Docker 운영은 컨테이너를 실행하는 것에서 끝나는 것이 아니라, 설정·데이터·상태·권한·자원을 분리하고 관리하는 것이다.**


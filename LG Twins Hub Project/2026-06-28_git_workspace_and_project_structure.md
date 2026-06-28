# 2026-06-28 Git 작업 환경 및 프로젝트 구조 정리

## 오늘 배운 핵심

오늘은 Git 명령어보다 **프로젝트를 어디에서 관리해야 하는지**에 대해 이해하는 시간을 가졌다.

---

# 현재 작업 환경

현재는 Windows와 Rocky Linux VM을 함께 사용하고 있다.

## Windows

* VS Code 사용
* OneDrive 공유 폴더 사용
* study-Linux 프로젝트 관리

## Rocky Linux VM

* Linux 실습
* Git 실습
* lg-twins-hub 프로젝트 개발

## GitHub

GitHub는 인터넷에 있는 원격 저장소이며

Windows와 Rocky Linux 모두 GitHub와 연결할 수 있다.

---

# 현재 프로젝트 역할

## study-Linux

공부 기록 저장소

관리할 내용

* Linux
* Git
* Nginx
* FastAPI
* Docker
* AWS
* 실습 정리
* Markdown 문서

공부한 내용은 모두 이 저장소에서 관리한다.

---

## lg-twins-hub

프로젝트 저장소

관리할 내용

* Python 코드
* FastAPI
* HTML / CSS
* JavaScript
* Nginx 설정
* Docker
* requirements.txt
* 프로젝트 README
* docs(프로젝트 설계 문서)

실제 서비스를 만드는 코드와 프로젝트 관련 파일만 관리한다.

---

# 작업 원칙

앞으로는 저장소의 역할을 명확하게 분리한다.

### study-Linux

* 공부
* 실습
* 학습 기록
* 명령어 정리

### lg-twins-hub

* 프로젝트 코드
* 기능 개발
* 프로젝트 문서
* 배포 관련 설정

---

# 오늘 이해한 Git 구조

Git은 프로젝트의 변경 이력을 관리하는 도구이다.

프로젝트마다 하나의 Git 저장소(.git)를 가진다.

```text
study-Linux
    └── .git

lg-twins-hub
    └── .git
```

각 프로젝트는 서로 독립적으로 관리된다.

---

# 앞으로의 개발 흐름

1. Linux, Git, Docker 등을 공부한다.
2. study-Linux 저장소에 학습 내용을 Markdown으로 정리한다.
3. 프로젝트 기능을 개발한다.
4. lg-twins-hub 저장소에서 Commit 및 Push를 진행한다.

공부와 프로젝트를 분리하면 저장소의 목적이 명확해지고, 나중에 포트폴리오를 만들 때도 훨씬 관리하기 쉬워진다.

---

# 오늘 느낀 점

오늘은 Git보다 작업 환경을 이해하는 데 많은 시간을 사용했다.

Windows, OneDrive, VMware, Rocky Linux, GitHub가 각각 어떤 역할을 하는지 이해하기 시작했고, 앞으로는 프로젝트와 공부를 분리해서 관리하기로 결정했다.

이 구조는 앞으로 Docker, AWS, Kubernetes를 공부할 때도 동일한 개념으로 이어질 것이다.

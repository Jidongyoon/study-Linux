# 📚 2026-08-07 학습 기록

## 🏫 오전 — 국비 교육

### 🖥️ 컴퓨터 시스템 & Linux

컴퓨터 시스템과 서버 인프라를 구성하는 기본 요소들을 학습하고, Linux 환경에서 시스템 상태를 확인하고 문제를 진단하는 기본 흐름을 익혔다.

- CPU, Memory, Disk, Process 등 시스템 주요 자원의 역할 이해
- `top`, `free`, `df`, `du` 등을 활용한 시스템 자원 확인
- Load Average를 통해 CPU 및 시스템 부하를 판단하는 방법 학습
- Memory의 `available` 값을 통해 실제 메모리 여유 상태를 확인하는 방법 학습
- `df -h`를 통한 디스크 용량 확인과 `df -i`를 통한 inode 사용량 확인
- `dmesg`, `journalctl` 등을 활용하여 시스템 및 커널 로그 확인
- 장애 발생 시 무작정 재시작하기보다 시스템 상태 → 자원 사용량 → 로그 순서로 확인하며 원인을 좁혀가는 과정 이해

### 🔍 시스템 장애 분석 흐름

서비스 상태 확인 → CPU / Process 확인 → Memory 확인 → Disk 확인 → Log 확인 → 원인 분석 및 조치

단순히 Linux 명령어를 외우는 것이 아니라, 각 명령어가 실제 장애 상황에서 어떤 정보를 확인하기 위해 사용되는지 중심으로 학습했다.

---

## 💻 오후 — 자율 학습

### 🐳 Docker

Mac 환경에서 Docker를 직접 설치하고 Ubuntu 24.04 컨테이너를 실행하며 Docker의 기본 동작 방식을 실습했다.

- `docker run -it ubuntu:24.04`를 이용하여 Ubuntu 컨테이너 실행
- 컨테이너 내부에서 `pwd`, `ls`, `/etc/os-release`, `uname -a` 등을 실행하여 환경 확인
- `docker ps`와 `docker ps -a`를 이용하여 실행 중인 컨테이너와 종료된 컨테이너 확인
- `docker start`, `docker stop`을 이용한 컨테이너 상태 관리
- 컨테이너 내부에서 `docker ps`를 실행했을 때 `docker: command not found`가 발생하는 것을 확인
- 이를 통해 Docker를 관리하는 호스트 환경과 컨테이너 내부 환경이 서로 다른 환경이라는 점을 이해

### 🐧 Linux 기본 명령어 실습

Docker Ubuntu 컨테이너 내부에서 파일과 디렉터리를 직접 생성하고 이동하면서 Linux 기본 명령어를 실습했다.

- `pwd` : 현재 작업 디렉터리 확인
- `ls` : 파일 및 디렉터리 목록 확인
- `cd` : 디렉터리 이동
- `cp` : 파일 또는 디렉터리 복사
- `mv` : 파일 또는 디렉터리 이동 및 이름 변경
- `./` : 현재 디렉터리
- `../` : 부모 디렉터리

특히 `cp`와 `mv`를 직접 사용하고 상대경로를 이용해 파일을 이동해보면서 Linux 파일 시스템의 경로 개념을 익혔다.

---

## 🍎 Mac Terminal 환경 구성

새로운 Mac 환경에서 인프라 공부를 계속하기 위해 터미널과 개발 환경을 직접 구성했다.

### Homebrew

Homebrew를 설치하고 Mac에서 패키지를 관리하는 방법을 익혔다.

운영체제별 패키지 관리 방식도 비교해보았다.

- Ubuntu → `apt`
- Rocky Linux → `dnf`
- macOS → `Homebrew`

### Git

Git을 설치하고 GitHub에서 사용할 사용자 이름과 이메일을 설정하는 방법을 확인했다.

앞으로 국비 교육에서 배운 내용과 개인 프로젝트를 GitHub에 지속적으로 기록하고 관리할 수 있도록 기본 환경을 구성했다.

### Mac Terminal

Mac의 터미널 환경을 직접 사용하면서 Linux에서 익숙하게 사용했던 CLI 환경과 비교해보았다.

운영체제는 다르지만 터미널을 통해 파일 관리, 개발 도구 설치, Git, Docker 등의 작업을 수행할 수 있다는 것을 경험했다.

---

## 💡 오늘의 핵심

오늘은 오전 국비 교육을 통해 컴퓨터 시스템과 Linux 서버의 기본적인 구조와 장애 분석 방법을 학습하고, 오후에는 자율적으로 Docker와 Linux 명령어를 실습하면서 Mac 환경까지 직접 구성해보았다.

특히 다음과 같은 인프라 학습의 큰 흐름을 연결해서 이해할 수 있었다.

Computer System → Linux / Server → Docker / Container → Cloud Infrastructure

명령어를 단순히 암기하는 것보다 **왜 사용하는지 이해하고 → 직접 실행해보고 → 문제가 발생하면 원인을 확인하는 과정**이 중요하다는 것을 느꼈다.

---

## 🚀 다음 학습 목표

- Docker Image와 Container 구조 이해
- Dockerfile 작성 및 이미지 빌드
- Docker Network / Volume 실습
- Linux 시스템 장애 분석 능력 보강
- Kubernetes 기본 개념 학습
- Linux → Docker → Kubernetes → Cloud로 이어지는 전체 인프라 흐름 이해

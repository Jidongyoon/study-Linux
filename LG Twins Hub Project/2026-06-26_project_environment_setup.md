# Project Environment Setup (2026-06-26)

## 프로젝트 목표

LG Twins Hub 프로젝트를 시작하기 위한 개발 환경을 구축하고, FastAPI 기반 웹 서비스를 개발하기 위한 기본 구조를 설계하였다.

---

# 개발 환경 점검

## 운영체제 확인

```bash
cat /etc/os-release
```

* Rocky Linux 9.0 (Blue Onyx)

---

## 커널 확인

```bash
uname -r
```

* Linux Kernel 5.14

---

## 메모리 확인

```bash
free -h
```

확인 내용

* Memory : 약 4GB
* Swap : 4GB

---

## 디스크 확인

```bash
df -h
```

확인 내용

* Root(/) : 76GB
* 사용률 : 약 8%

---

## Hostname 확인

```bash
hostnamectl
```

확인 내용

* VMware Virtual Platform
* Rocky Linux 9.0

---

## 네트워크 확인

```bash
ip addr
```

확인 내용

* Interface : ens160
* IP : 192.168.111.100

---

## 시간 동기화 확인

```bash
date
timedatectl
```

확인 내용

* Time Zone : Asia/Seoul
* NTP Service : Active
* System Clock : Synchronized

---

## 사용자 확인

```bash
whoami
id
pwd
```

확인 내용

* 일반 사용자(rocky) 계정 사용
* wheel 그룹 포함
* 작업 디렉터리 : /home/rocky

---

## 네트워크 연결 확인

```bash
ping -c 3 8.8.8.8
ping -c 3 www.google.com
```

확인 내용

* 인터넷 연결 정상
* DNS 정상 동작

---

# 프로젝트 구조 설계

프로젝트 생성

```text
lg-twins-hub/
├── app/
├── config/
├── docs/
├── logs/
├── nginx/
├── scripts/
├── static/
├── templates/
├── tests/
├── README.md
└── .gitignore
```

### 각 디렉터리 역할

* app : FastAPI 애플리케이션
* config : 환경 설정
* docs : 프로젝트 문서
* logs : 로그 관리
* nginx : Nginx 설정
* scripts : 관리 스크립트
* static : CSS, JavaScript, 이미지
* templates : HTML 템플릿
* tests : 테스트 코드

---

# Python 개발 환경 준비

Python 프로젝트는 프로젝트별 독립적인 라이브러리 관리를 위해 `venv`(Virtual Environment)를 사용한다.

학습 내용

* 프로젝트별 Python 환경 분리
* 라이브러리 버전 충돌 방지
* requirements.txt를 통한 환경 공유
* GitHub에는 venv를 업로드하지 않음

---

# .gitignore 구성

```text
venv/
__pycache__/
*.pyc
.vscode/
.DS_Store
*.log
logs/
```

학습 내용

* Git에서 관리하지 않을 파일 지정
* 가상환경 및 캐시 파일 제외
* 로그 파일 제외

---

# 오늘 학습한 핵심 개념

* 프로젝트 구조를 먼저 설계하는 이유
* 역할별 디렉터리 분리
* FastAPI 프로젝트 구조 이해
* GitHub 관리 방식
* Python Virtual Environment(venv) 이해
* .gitignore의 역할
* 서버 환경 사전 점검 절차

---

# 다음 목표

* Python Virtual Environment 활성화
* FastAPI 설치
* Uvicorn 설치
* requirements.txt 생성
* FastAPI Hello World 구현
* Git Commit

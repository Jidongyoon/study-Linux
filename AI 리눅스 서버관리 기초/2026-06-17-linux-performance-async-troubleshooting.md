# Linux 성능 분석, 비동기 처리, 트러블슈팅 정리

## 개요

이번 실습에서는 Linux 성능 분석, 프로세스와 스레드, Load Average, subprocess, Redis Lock, 비동기 처리, 그리고 운영자 관점의 트러블슈팅 방법을 학습했다.

---

## Load Average

Load Average는 CPU 사용률이 아니라 CPU를 사용하려고 실행 중이거나 대기 중인 작업의 수를 의미한다.

예를 들어 CPU 코어가 2개인 서버에서 Load Average가 24라면 CPU가 처리 가능한 수준보다 훨씬 많은 작업이 대기 중인 상태라고 볼 수 있다.

### 관련 개념

### Run Queue

CPU를 사용하기 위해 대기하는 작업 목록

예시

- CPU 2개
- 작업 40개

실제로는 2개만 실행되고 나머지는 Run Queue에서 대기한다.

### Wait Queue

특정 이벤트를 기다리는 작업 목록

예시

- 디스크 I/O 완료 대기
- 네트워크 응답 대기

---

## 프로세스와 스레드

### 프로세스(Process)

실행 중인 프로그램

예시

```bash
python
git
uvicorn
```

### 스레드(Thread)

프로세스 내부에서 실행되는 작업 단위

예를 들어 FastAPI 서버는 하나의 프로세스 안에서 여러 스레드를 생성하여 작업을 수행할 수 있다.

---

## R 상태와 D 상태

### R (Running)

- CPU 실행 중
- CPU 할당 대기 중

### D (Disk Sleep)

- 디스크 I/O 완료 대기 상태

### 해석

- R 상태가 많으면 CPU 병목
- D 상태가 많으면 디스크 I/O 병목

---

## sysbench를 이용한 CPU 부하 실험

```bash
sysbench cpu --threads=1 --time=60 run
sysbench cpu --threads=2 --time=60 run
sysbench cpu --threads=40 --time=60 run
```

실험 목적

- 스레드 수에 따른 CPU 사용률 변화 확인
- Load Average 변화 확인
- Run Queue 증가 확인

관찰 결과

- CPU 코어 수보다 스레드 수가 많다고 무조건 성능이 좋아지는 것은 아니다.
- CPU는 계속 작업을 처리하지만 대기열이 증가하여 응답 속도가 느려질 수 있다.

---

## subprocess 이해

Python에서 외부 프로그램을 실행할 때 사용한다.

예시

```python
subprocess.run(["git", "clone", github_url])
```

동작 구조

```text
Python
  ↓
subprocess
  ↓
git clone
```

즉, Python이 직접 Git 기능을 구현하는 것이 아니라 Git 프로그램을 실행시키는 방식이다.

---

## 비동기 처리와 Redis Lock

### 기존 방식

```text
요청
 ↓
git clone
 ↓
cloc
 ↓
응답
```

문제점

- 사용자가 오랫동안 대기
- 동일 요청이 여러 번 실행될 수 있음

### 개선 방식

```text
요청
 ↓
Thread 생성
 ↓
processing 응답
 ↓
백그라운드 분석
 ↓
Redis 저장
 ↓
완료 응답
```

### Redis Lock

동일 저장소에 대한 중복 분석을 방지하기 위해 사용

예시

```text
analysis:progress:pytorch/pytorch
```

이미 Lock이 존재하면 새로운 작업을 생성하지 않고 processing 상태를 반환한다.

---

## Git Clone 인터랙티브 문제

잘못된 저장소 URL을 입력했을 때 다음과 같은 상황이 발생했다.

```bash
git clone https://github.com/pytorch/pytor.git
```

결과

```text
Username for 'https://github.com':
```

Git이 사용자 입력을 기다리는 인터랙티브 상태에 들어갔다.

서버 환경에서는 입력할 사용자가 없기 때문에 작업이 영원히 종료되지 않을 수 있다.

### 해결 방법

```bash
GIT_TERMINAL_PROMPT=0
```

예시

```bash
GIT_TERMINAL_PROMPT=0 git clone https://github.com/pytorch/pytor.git
```

결과

```text
fatal: could not read Username for 'https://github.com'
terminal prompts disabled
```

즉시 실패(Fail Fast)하여 무한 대기를 방지할 수 있다.

---

## subprocess 타임아웃 처리

외부 프로그램이 너무 오래 실행되는 경우를 방지하기 위해 timeout 옵션을 사용할 수 있다.

예시

```python
subprocess.run(
    cmd,
    timeout=10,
    check=True,
)
```

10초 이상 실행되면 TimeoutExpired 예외가 발생한다.

이를 통해 무한 대기 상황을 예방할 수 있다.

---

## 운영자(슈퍼유저) 관점의 트러블슈팅

문제가 발생하면 코드부터 수정하지 않는다.

먼저 현재 시스템 상태를 확인한다.

주요 명령어

```bash
ps -ef
top
htop
iotop
cat /proc/loadavg
```

트러블슈팅 순서

```text
증상 확인
 ↓
프로세스 확인
 ↓
로그 확인
 ↓
실제 실행 명령 확인
 ↓
원인 분석
 ↓
수정
```

실습 사례

```text
분석이 진행중입니다
 ↓
ps -ef | grep git
 ↓
git clone 확인
 ↓
URL 오타 발견
 ↓
문제 해결
```

### 핵심 교훈

추측보다 관측이 우선이다.

코드만 보지 말고 현재 시스템에서 실제로 어떤 프로세스가 실행되고 있는지 확인하는 습관이 중요하다.

---

## 정리

- Load Average는 CPU 사용률이 아니라 작업 대기 상태를 의미한다.
- Run Queue는 CPU 대기열이다.
- R 상태는 CPU 병목, D 상태는 I/O 병목을 의미한다.
- subprocess는 Python이 외부 프로그램을 실행하는 방법이다.
- Redis Lock은 중복 작업을 방지한다.
- 인터랙티브 명령은 서버 환경에서 위험할 수 있다.
- timeout과 GIT_TERMINAL_PROMPT=0은 안전한 서버 운영에 도움이 된다.
- 트러블슈팅은 추측보다 관측이 우선이다.
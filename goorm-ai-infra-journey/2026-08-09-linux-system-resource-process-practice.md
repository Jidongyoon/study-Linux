# Linux 시스템 자원 · 프로세스 · 로그 분석 실습

> 2026-08-09 학습 기록

## 1. CPU와 Load Average

### uptime

    uptime

시스템의 가동 시간과 Load Average를 확인한다.

예시:

    uptime
    08:35:30 up 16 min, 0 users, load average: 5.06, 1.46, 0.50

### top

    top

주요 항목:

- us: 사용자 영역 CPU 사용률
- sy: 커널 영역 CPU 사용률
- ni: nice로 조정된 프로세스의 CPU 사용률
- id: CPU 유휴 시간
- wa: I/O 대기 시간
- hi: 하드웨어 인터럽트
- si: 소프트웨어 인터럽트
- st: 가상화 환경에서 다른 VM에 의해 빼앗긴 CPU 시간

실습에서 여러 개의 bash 프로세스를 실행하여 CPU 사용률과 Load Average가 증가하는 것을 확인했다.

CPU 사용률과 Load Average는 같은 개념이 아니다.

- CPU 사용률: 현재 CPU가 얼마나 사용되고 있는지 보여주는 지표
- Load Average: CPU에서 실행 중이거나 실행을 기다리는 작업 등의 시스템 부하를 나타내는 지표

CPU가 100% 사용되는 상황에서 Load Average가 함께 증가하는 것을 직접 확인했다.

---

## 2. 메모리 확인

### free

    free -h

주요 항목:

- total: 전체 메모리
- used: 사용 중인 메모리
- free: 현재 사용되지 않는 메모리
- shared: 공유 메모리
- buff/cache: 버퍼와 캐시
- available: 새로운 프로세스에 할당할 수 있는 메모리
- Swap: 스왑 사용량

예시:

    Mem: 7.8Gi 495Mi 6.7Gi 0.0Ki 692Mi 7.3Gi
    Swap: 1.0Gi 0B 1.0Gi

### 메모리 사용량 증가 실습

    python3 -c "a = bytearray(1024*1024*1024); input()"

약 1GiB의 메모리를 할당한 상태로 프로세스를 유지한 후 free와 top으로 메모리 변화를 확인했다.

top에서 해당 Python 프로세스의 RES가 약 1GiB까지 증가하는 것을 확인했다.

메모리는 프로세스가 종료되면 해당 프로세스가 사용하던 메모리가 다시 회수된다.

---

## 3. Docker 메모리 제한과 OOM

Docker 컨테이너에 메모리 제한을 설정했다.

    docker run -it --name rocky-oom --memory=1g rockylinux:9 bash

컨테이너 내부에서 메모리 제한을 확인했다.

    cat /sys/fs/cgroup/memory.max

결과:

    1073741824

약 1GiB의 메모리 제한이 설정된 것을 확인했다.

### 현재 메모리 사용량 확인

    cat /sys/fs/cgroup/memory.current

### 메모리 이벤트 확인

    cat /sys/fs/cgroup/memory.events

OOM 상황에서 다음과 같은 값을 확인했다.

    low 0
    high 0
    max 553
    oom 2
    oom_kill 2
    oom_group_kill 0

메모리 제한을 초과하면서 OOM(Out Of Memory)이 발생하고 프로세스가 OOM Kill되는 것을 직접 확인했다.

### 핵심 흐름

    컨테이너 메모리 제한
            ↓
    메모리 사용량 증가
            ↓
    메모리 제한에 접근
            ↓
    OOM 발생
            ↓
    OOM Kill
            ↓
    프로세스 종료

---

## 4. Docker 컨테이너와 메모리

기존 Rocky Linux 컨테이너의 메모리 제한을 확인했다.

    docker inspect rocky-mission --format '{{.HostConfig.Memory}}'

결과:

    0

이는 Docker 컨테이너 자체에 별도의 메모리 제한이 설정되어 있지 않다는 의미다.

컨테이너 내부에서는:

    cat /sys/fs/cgroup/memory.max

결과:

    max

Docker Desktop 환경에서는 컨테이너가 Docker Desktop에서 할당받은 자원을 사용하는 구조이므로 호스트의 메모리와 컨테이너 자원 제한 관계를 이해하는 것이 중요하다.

---

## 5. 디스크와 inode

### 디스크 용량 확인

    df -h

예시:

    Filesystem      Size  Used Avail Use% Mounted on
    overlay         453G  1.6G  428G   1% /

df -h는 파일 시스템의 전체 용량, 사용량, 남은 용량, 사용률 등을 사람이 읽기 쉬운 단위로 보여준다.

### inode 확인

    df -i

예시:

    Filesystem       Inodes IUsed    IFree IUse% Mounted on
    overlay        30179328 11918 30167410    1% /

inode는 파일 시스템에서 파일과 디렉터리의 메타데이터를 관리하는 구조다.

디스크 공간이 남아 있어도 inode를 모두 사용하면 새로운 파일을 생성하지 못할 수 있다.

즉 디스크 문제를 확인할 때는 용량뿐 아니라 inode 사용량도 함께 확인해야 한다.

---

## 6. 서버 자원 점검 스크립트

서버의 주요 자원을 한 번에 확인하기 위해 check_server.sh를 작성했다.

    #!/bin/bash

    echo "===== 서버 점검 ====="

    echo
    echo "[CPU]"
    echo "CPU Cores: $(nproc)"
    echo "Load Average: $(uptime | awk -F'load average: ' '{print $2}')"

    echo
    echo "[Memory]"
    free -h

    echo
    echo "[Disk]"
    df -h /

    echo
    echo "[Inode]"
    df -i /

실행 권한 부여:

    chmod +x check_server.sh

실행:

    ./check_server.sh

CPU, Load Average, Memory, Disk, Inode를 한 번에 확인할 수 있도록 구성했다.

---

## 7. 프로세스 확인과 종료

백그라운드 프로세스를 생성했다.

    sleep 300 &

프로세스 확인:

    ps -ef | grep sleep

PID를 확인한 후 해당 프로세스를 종료했다.

    kill <PID>

다시 확인:

    ps -ef | grep sleep

sleep 프로세스가 사라진 것을 확인했다.

---

## 8. Foreground / Background 프로세스 제어

백그라운드 실행:

    sleep 600 &

작업 확인:

    jobs

Foreground로 전환:

    fg %1

실행 중인 프로세스를 일시 정지:

    Ctrl + Z

정지 상태 확인:

    jobs

다시 Background로 실행:

    bg %1

최종적으로 다음과 같은 프로세스 상태 전환을 실습했다.

    Background
        ↓
    Foreground
        ↓
    Stopped
        ↓
    Background

### 명령어 정리

- jobs: 현재 셸에서 관리 중인 작업 확인
- fg: 백그라운드 작업을 포그라운드로 전환
- bg: 정지된 작업을 백그라운드에서 다시 실행
- Ctrl + Z: 현재 포그라운드 작업을 정지
- kill: 프로세스 종료 신호 전달

---

## 9. 로그 분석

테스트용 app.log 파일을 생성하고 ERROR 로그를 분석했다.

### ERROR 로그만 추출

    grep "ERROR" app.log

ERROR가 포함된 로그만 추출할 수 있다.

### ERROR 개수 확인

    grep -c "ERROR" app.log

결과:

    5

총 5개의 ERROR 로그가 존재하는 것을 확인했다.

### ERROR 뒤의 메시지만 추출

    grep "ERROR" app.log | sed 's/.*ERROR //'

결과:

    Connection failed
    Connection failed
    Timeout
    Connection failed
    Timeout

### 오류 유형별 발생 횟수 집계

    grep "ERROR" app.log | sed 's/.*ERROR //' | sort | uniq -c

결과:

    3 Connection failed
    2 Timeout

### 명령어 역할

grep
- 특정 문자열이나 패턴을 검색한다.

sed
- 텍스트를 가공하고 불필요한 문자열을 제거한다.

sort
- 입력된 줄을 정렬한다.
- 같은 문자열을 서로 붙여서 정렬할 수 있다.

uniq -c
- 연속해서 반복되는 중복 줄을 하나로 묶고 개수를 표시한다.
- 따라서 일반적으로 sort와 함께 사용한다.

전체 파이프라인:

    grep
      ↓
    ERROR 로그 추출
      ↓
    sed
      ↓
    오류 메시지만 추출
      ↓
    sort
      ↓
    같은 오류끼리 정렬
      ↓
    uniq -c
      ↓
    오류 유형별 발생 횟수 집계

---

## 10. 환경 변수

환경 변수는 프로세스가 실행될 때 전달할 수 있는 설정값이다.

환경 변수 생성:

    export APP_ENV=production

값 확인:

    echo $APP_ENV

결과:

    production

환경 변수 목록에서 확인:

    env | grep APP_ENV

결과:

    APP_ENV=production

### export의 의미

    export APP_ENV=production

export는 현재 셸의 변수를 환경 변수로 만들어 자식 프로세스에 전달할 수 있도록 한다.

자식 Bash를 실행하여 환경 변수 상속을 확인했다.

    bash

자식 Bash에서:

    echo $APP_ENV

결과:

    production

이를 통해 부모 프로세스의 환경 변수가 자식 프로세스에 상속되는 것을 확인했다.

### 환경 변수의 활용

환경별 설정을 코드에 직접 하드코딩하지 않고 실행 환경에서 전달할 수 있다.

예:

    APP_ENV=development

개발 환경에서는 development를 사용하고,

    APP_ENV=production

운영 환경에서는 production을 사용하는 방식으로 코드와 환경 설정을 분리할 수 있다.

---

## 11. 서비스 전용 계정 구성

초기 구성 자동화 미션을 진행하면서 서비스 전용 계정을 생성했다.

### 서비스 디렉터리 생성

    mkdir -p /opt/myapp

mkdir의 -p 옵션은 필요한 부모 디렉터리를 함께 생성하고, 이미 존재하는 디렉터리도 오류 없이 처리할 수 있도록 한다.

### 서비스 전용 계정 생성

    useradd -r -s /sbin/nologin myapp

확인:

    id myapp

결과:

    uid=999(myapp) gid=999(myapp) groups=999(myapp)

### 옵션 의미

-r
- 시스템 계정으로 생성한다.
- 사람이 직접 사용하는 일반 계정보다 서비스 실행용 계정에 적합하다.

-s /sbin/nologin
- 해당 계정이 일반적인 로그인 쉘을 사용하지 못하도록 설정한다.

서비스를 root 계정으로 실행하지 않고 별도의 서비스 계정으로 실행하면 권한을 분리할 수 있다.

### 다음 실습 예정

현재 /opt/myapp은 root 소유이므로 다음 단계에서 서비스 계정인 myapp이 사용할 수 있도록 소유권을 변경할 예정이다.

    chown myapp:myapp /opt/myapp

---

## 12. 이번 실습에서 정리한 핵심 명령어

    uptime
    top
    free -h
    df -h
    df -i
    ps -ef
    kill
    jobs
    fg
    bg
    grep
    sed
    sort
    uniq -c
    export
    env
    mkdir -p
    useradd
    id
    chown

---

## 13. 학습 정리

이번 실습에서는 Linux 서버 운영에서 자주 확인하는 주요 자원을 직접 확인하고 부하 상황을 만들어보았다.

CPU에서는 CPU 사용률과 Load Average의 차이를 확인했고,
메모리에서는 free와 top을 이용해 메모리 사용량을 확인했다.

Docker에서는 cgroup을 통해 컨테이너의 메모리 제한을 확인하고 실제 OOM 및 OOM Kill 상황까지 재현했다.

디스크에서는 df -h와 df -i를 이용해 디스크 용량과 inode 사용량을 확인했다.

프로세스 실습에서는 ps, kill, jobs, fg, bg를 사용하여 프로세스의 실행 상태와 백그라운드 작업을 직접 제어했다.

로그 분석에서는 grep, sed, sort, uniq -c를 조합하여 오류 메시지를 추출하고 유형별 발생 횟수를 집계했다.

마지막으로 환경 변수와 서비스 전용 계정을 직접 구성하면서 단순한 명령어 사용을 넘어 실제 서버 초기 구성과 운영에 필요한 개념을 연결해보았다.
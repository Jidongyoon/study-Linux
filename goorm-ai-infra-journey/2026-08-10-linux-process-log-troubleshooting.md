# 2026-08-10 Linux 서버 / 프로세스 / 로그 / 장애 분석

## 기술 스택

- Linux / Ubuntu 24.04
- Process Management
- systemd
- journald / journalctl
- Docker
- Python
- cgroup
- Memory / Disk Monitoring

---

## 오늘 배운 것

오늘은 Linux 서버에서 실행되는 프로세스와 서비스의 상태를 확인하고, 로그를 이용해 장애 원인을 추적하는 방법을 학습했다.

### 서버 자원 확인

Linux 서버의 기본적인 상태를 확인하기 위해 CPU, Memory, Disk 상태를 직접 점검했다.

- `nproc` : CPU 코어 수 확인
- `uptime` : Load Average 확인
- `top` : CPU / Memory / Process 상태 확인
- `free -h` : Memory 상태 확인
- `df -h` : Disk 사용량 확인
- `df -i` : inode 사용량 확인

Load Average는 1분, 5분, 15분 기준으로 확인하고 CPU 코어 수와 비교해 현재 서버의 부하 상태를 판단할 수 있다.

메모리는 단순히 `free` 값만 보는 것이 아니라 `available` 값을 통해 실제로 사용할 수 있는 메모리 여유를 확인하는 것이 중요하다는 것을 배웠다.

디스크는 전체 용량뿐만 아니라 inode가 부족한 경우에도 파일 생성 등에 문제가 발생할 수 있기 때문에 `df -h`와 `df -i`를 각각 확인해야 한다.

### Process

Linux에서 실행 중인 프로세스를 확인하면서 PID, PPID, CPU 사용량, Memory 사용량, Process State 등의 정보를 확인했다.

프로세스 상태값을 통해 현재 프로세스가 실행 중인지 대기 중인지 판단하고, 어떤 프로세스가 서버의 자원을 많이 사용하고 있는지도 확인했다.

장애 상황에서는 서비스만 확인하는 것이 아니라 실제로 어떤 프로세스가 실행되고 있고 어떤 자원을 사용하고 있는지 함께 확인해야 한다는 것을 배웠다.

### Systemd

systemd를 이용해 서비스를 관리하고 서비스가 실패하는 상황을 직접 실습했다.

주요 명령어:

- `systemctl status`
- `systemctl start`
- `systemctl restart`
- `systemctl show`
- `systemctl cat`

Unit 파일의 설정을 확인하면서 서비스가 어떤 프로그램을 실행하는지 확인했다.

주요 설정:

- `ExecStart` : 실행할 프로그램 지정
- `Restart=on-failure` : 프로세스가 실패하면 재시작
- `RestartSec=2` : 재시작 전 대기 시간

`Restart=on-failure`가 설정되어 있으면 프로그램이 오류로 종료되어도 systemd가 다시 실행하기 때문에 서비스가 계속 종료되고 다시 시작되는 Restart Loop가 발생할 수 있다.

### journald / journalctl

systemd에서 관리하는 서비스의 로그를 확인하는 방법을 배웠다.

서비스가 출력하는 stdout과 stderr를 systemd가 받아 journald에 기록하고 `journalctl`을 이용해 해당 로그를 조회할 수 있다.

주요 명령어:

- `journalctl -u 서비스명`
- `journalctl -u 서비스명 -n 10 --no-pager`
- `journalctl -u 서비스명 -n 50`

서비스 장애가 발생했을 때 `systemctl status`만 확인하는 것이 아니라 `journalctl`을 통해 실제로 어떤 오류가 발생했는지 확인해야 한다.

로그에서 `Failed`, `RuntimeError`, `exit-code`, `status` 등의 메시지를 확인하면서 프로세스가 종료되기 직전의 로그가 장애 원인을 찾는 중요한 단서가 된다는 것을 배웠다.

### Restart Loop 장애 분석

서비스가 시작된 후 프로그램이 오류로 종료되고 systemd가 다시 실행하는 상황을 직접 확인했다.

서비스가 계속 재시작되면 `NRestarts` 또는 로그의 restart counter를 통해 반복적인 재시작이 발생하고 있는지 확인할 수 있다.

서비스 상태가 `active`로 표시되고 있더라도 시작 시간이 계속 변경되거나 재시작 횟수가 증가한다면 정상적인 상태가 아닐 수 있다는 것을 배웠다.

### 서비스 실행 파일 경로 문제 분석

실습 중 systemd 서비스가 정상적으로 시작되지 않는 장애를 확인했다.

로그에서 다음과 같은 내용을 확인했다.

`python3: can't open file '/home/stu01/app.py': [Errno 2] No such file or directory`

이를 통해 Python 자체의 문제가 아니라 `ExecStart`에서 지정한 파일 경로에 실제 파일이 존재하지 않는 것이 원인이라는 것을 로그를 통해 확인했다.

또한 `status=2/INVALIDARGUMENT`와 같은 종료 상태도 확인하면서 단순히 에러 코드만 보는 것이 아니라 그 앞뒤의 로그를 함께 확인해야 실제 원인을 찾을 수 있다는 것을 배웠다.

### Docker Memory / OOM

Docker 컨테이너의 메모리를 제한하고 실제 OOM(Out Of Memory) 상황을 재현했다.

컨테이너 실행 시 `--memory=512m` 옵션을 사용해 메모리 사용량을 512MB로 제한했다.

컨테이너 내부에서 cgroup 설정을 확인했다.

- `/sys/fs/cgroup/memory.max`
- `/sys/fs/cgroup/memory.swap.max`

메모리를 계속 사용하는 Python 프로그램을 실행해 제한된 메모리를 초과하도록 만들었고 프로세스가 `Killed`되는 상황을 확인했다.

프로세스 종료 후 종료 코드 `137`을 확인했고 Docker에서 `OOMKilled=true`를 확인해 실제 메모리 부족으로 인해 프로세스가 강제 종료된 것을 검증했다.

### Memory / Swap

메모리 부족 상황을 분석하기 위해 RAM뿐만 아니라 Swap 상태도 함께 확인했다.

확인한 명령어:

- `free -h`
- `swapon --show`
- `/proc/meminfo`
- `cat /sys/fs/cgroup/memory.max`
- `cat /sys/fs/cgroup/memory.swap.max`

이를 통해 실제 서버나 컨테이너의 메모리 문제를 확인할 때 RAM 사용량뿐만 아니라 Swap과 cgroup 메모리 제한도 함께 확인해야 한다는 것을 배웠다.

---

## 자율 실습

강의 내용을 따라가는 것에서 끝내지 않고 직접 장애 상황을 만들고 원인을 확인하는 자율 실습도 진행했다.

서비스가 반복적으로 재시작되는 상황을 만들고 `systemctl status`와 `journalctl`을 이용해 장애 발생 시점의 로그를 확인했다.

또한 서버 자원을 직접 확인하면서 CPU, Memory, Disk 상태가 정상인지 판단하고 프로세스가 자원을 과도하게 사용하고 있는지도 확인했다.

Docker에서는 컨테이너의 메모리를 제한한 뒤 Python 프로세스를 이용해 OOM을 발생시키고 종료 코드와 `OOMKilled` 값을 통해 원인을 검증했다.

---

## 장애 분석 흐름

오늘 실습을 통해 다음과 같은 기본적인 장애 분석 흐름을 연습했다.

시스템 상태 확인
→ CPU / Memory / Disk 확인
→ Process 확인
→ Service 상태 확인
→ journalctl 로그 확인
→ 장애 발생 직전 로그 확인
→ 원인 추정
→ 설정 또는 자원 상태 확인
→ 문제 수정
→ 서비스 재실행
→ 로그와 상태를 통해 정상 여부 검증

---

## 오늘의 핵심

오늘은 단순히 Linux 명령어를 사용하는 것보다 각각의 명령어가 어떤 장애를 확인하기 위해 사용되는지 연결해서 이해하는 데 집중했다.

특히 Process → systemd → journald → journalctl → 장애 로그라는 흐름을 이해하면서 서비스 장애가 발생했을 때 로그를 기반으로 원인을 추적하는 방법을 익혔다.

또한 Docker에서 OOM을 직접 재현하면서 메모리 부족이 실제 프로세스 강제 종료로 이어지는 과정과 종료 코드 `137`, `OOMKilled=true`를 통해 원인을 검증하는 방법도 경험했다.

이번 실습을 통해 장애가 발생했을 때 바로 해결 방법부터 찾기보다 먼저 현재 상태를 확인하고 로그와 수치를 근거로 원인을 좁혀가는 것이 중요하다는 것을 배웠다.
 Linux System & Memory Engineering Deep Dive클라우드 엔지니어링의 기반이 되는 리눅스 커널의 메모리 관리 메커니즘과 시스템 동작 원리를 정리합니다.1. 가상 메모리(Virtual Memory)의 구조와 할당리눅스에서 프로세스는 실제 RAM 주소를 직접 보지 못하며, 커널이 생성한 가상 주소(Virtual Address) 공간에서 동작합니다.🏗️ 핵심 커널 구조체task_struct: 프로세스의 모든 상태를 담은 커널 내부의 '컨트롤 타워'.mm_struct: 프로세스가 소유한 메모리 맵의 총체.vma (Virtual Memory Area): 주소 공간을 성격별로 쪼갠 단위 (Stack, Heap, Code 등).💡 할당의 실제 (Lazy Allocation)malloc()이나 new를 호출해도 즉시 RAM이 할당되지 않습니다. 커널은 VMA 주소만 기록해두고, 실제로 데이터를 쓸 때(Access) 비로소 물리 메모리를 연결합니다.2. 페이지 폴트(Page Fault) 트러블슈팅페이지 폴트는 '에러'가 아니라, 가상 메모리를 물리 메모리로 연결하기 위한 커널의 정상적인 이벤트입니다.🔍 폴트의 종류Minor Page Fault: 메모리 어딘가엔 데이터가 있지만 매핑만 안 된 상태. (비교적 빠름)Major Page Fault: 디스크(SSD/HDD)에서 데이터를 읽어와야 하는 상태. (I/O 병목의 주원인)🛠️ 관련 명령어Bash# 특정 PID의 페이지 폴트 발생 횟수 확인 (minflt, majflt)
ps -o min_flt,maj_flt,cmd -p [PID]

# 시스템 전체의 페이지 폴트 통계 보기 (1초 간격)
sar -B 1
3. 리눅스 메모리 '진짜 여유 공간' 해석법free -h 명령어 결과에서 **buff/cache**와 **available**을 구분하는 것이 클라우드 운영의 핵심입니다.

📊 메모리 지표 상세

항목,설명
vsz (Virtual Size),프로세스에 할당된 가상 메모리 총량 (실제 사용량 아님)
rss (Resident Set),실제로 물리 RAM에 점유 중인 데이터 크기
buff/cache,디스크 I/O 가속을 위해 커널이 빌려 쓰고 있는 공간
available,새로운 프로세스를 위해 즉시 비워줄 수 있는 '진짜' 여유분

4. 파일 입출력(I/O)과 페이지 캐시(Page Cache)
리눅스는 한 번 읽은 파일을 메모리에 복사해두어 디스크 접근을 최소화합니다.

⚡ Hit vs Miss
Cache Hit: 필요한 파일 내용이 메모리에 있어 즉시 반환. (고성능)

Cache Miss: 디스크까지 다녀와야 함. (레이턴시 발생)

📂 주요 개념: Anonymous Page vs File-backed Page
Anonymous Page: stack/heap 처럼 디스크에 원본 파일이 없는 데이터. (부족 시 Swap 발생)

File-backed Page: 디스크의 실제 파일 내용을 복사한 데이터. (부족 시 그냥 비워도 됨)

5. ABI(Application Binary Interface)의 중요성
바이너리 실행 파일이 OS나 하드웨어와 소통하기 위한 **'기계적 통신 규약'**입니다.

약속 내용: 데이터 크기(32/64bit), 함수 호출 규약(Calling Convention), 시스템 콜 번호 등.

장점: ABI가 표준화되어 있으면, 특정 리눅스 환경에서 컴파일한 실행 파일을 다른 리눅스 배포판에서도 그대로 실행 가능(바이너리 호환성)합니다.

🛠️ 실무 엔지니어가 자주 쓰는 핵심 명령어 리스트

# 1. 프로세스별 메모리 맵(VMA) 상세 주소 확인
sudo cat /proc/[PID]/maps

# 2. 시스템 콜 호출 현황 추적 (어떤 파일 오픈하고 읽는지 확인)
sudo strace -p [PID] -e open,read,write

# 3. 현재 열려 있는 모든 파일 디스크립터(FD) 확인
lsof -p [PID]

# 4. 파일의 상세 정보(inode, 권한 등) 확인
stat [파일명]

# 5. 메모리를 강제로 비워 캐시 비우기 (테스트 용도)
sync; echo 3 | sudo tee /proc/sys/vm/drop_caches
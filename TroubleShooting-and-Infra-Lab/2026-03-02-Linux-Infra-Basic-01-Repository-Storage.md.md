[2026-03-02] Infrastructure Study Log: Linux Repository & Network Storage
오늘의 학습은 리눅스 시스템의 확장성과 네트워크를 통한 자원 공유의 핵심 원리를 이해하는 데 집중했습니다. 특히 예기치 못한 패키지 설치 에러를 해결하며 리포지토리 관리 능력을 배양했습니다.

🛠️ 1. Troubleshooting Report: EPEL Repository Issue
🔍 문제 현상 (Issue)
상황: Rocky Linux 9 환경에서 htop 설치를 위해 epel-release를 설치하려 했으나 에러 발생.

에러 메시지: Error: Unable to find a match: epel-release

원인 분석:

Rocky 9의 기본 리포지토리 목록에 EPEL 경로가 누락되었거나 활성화되지 않음.

EPEL이 의존하는 crb (Code Ready Builder) 리포지토리가 비활성 상태임.

💡 해결 방법 (Solution)
단순한 설치 명령어가 아닌, 리포지토리 구성을 강제로 업데이트하고 직접 경로를 지정하여 해결함.

CRB 리포지토리 활성화: dnf config-manager --set-enabled crb

EPEL 최신 RPM 직접 설치:

dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

메타데이터 갱신: dnf clean all && dnf makecache

결과 확인: dnf repolist 명령어로 epel 활성화 확인 후 htop 설치 완료.

📖 2. Today's Key Concepts
📦 EPEL (Extra Packages for Enterprise Linux)
정의: RHEL 계열(Rocky, Alma 등)의 안정성을 해치지 않으면서, 공식 저장소에 없는 고품질 추가 패키지를 제공하는 확장 저장소.

핵심: "공식 창고(BaseOS)에 없는 보너스 아이템 창고"로 이해.

🔌 Client-Server Model & Installation
서비스의 구성: 통신을 위해서는 서비스를 제공하는 Server와 요청하는 Client 도구가 양쪽에 필요함.

실습 사례:

NFS: 서버에는 nfs-utils가 동작해야 하며, 클라이언트 역시 이를 마운트하기 위한 도구가 깔려 있어야 함.

Windows: 'Windows 기능 켜기/끄기'를 통해 텔넷(Telnet)이나 NFS 클라이언트를 수동으로 활성화해야 리눅스 서버와 통신 가능.

📂 Understanding 'Path' (경로)
개념: 파일 시스템 내에서 특정 자원의 위치를 나타내는 주소.

설정 값 path = /share:

path: 공유할 대상의 위치를 지정하는 키워드.

/share: 루트 디렉터리 바로 아래의 share 폴더라는 절대 경로.

주의: 설정 파일의 path와 실제 물리적 폴더가 일치하지 않으면 서비스 구동 실패.

🔐 Linux Permissions & Root
권한 철학: 시스템 전체에 영향을 주는 설정 파일 수정은 오직 **Root(집주인)**만 가능.

권한 대행:

su -c 'command': 특정 명령어 하나만 관리자 권한으로 실행.

sudo: 관리자 권한을 일시적으로 빌려 작업 (실무 권장 방식).

🚀 3. Practical Commands Summary

명령어,용도

dnf repolist,현재 활성화된 저장소(창고) 목록 확인
dnf makecache,저장소의 패키지 목록을 최신화 (속도 향상)
mount -t nfs [IP]:[Path] [Target],원격 서버의 폴더를 내 로컬 폴더처럼 연결
df -h,현재 마운트된 디스크 및 네트워크 스토리지 확인
pwd,현재 내가 서 있는 위치(Path) 확인
htop,"시스템 자원(CPU, RAM) 사용량을 보여주는 인터랙티브 모니터링 도구"

Tomorrow's Goals
[ ] NFS 마운트된 폴더 내 파일 생성 및 권한 테스트 (Read/Write)

[ ] vsftpd를 활용한 파일 서버 구축 및 윈도우 클라이언트 접속 테스트

### 🔄 Relationship between Server and Client

- **Server-side**: 서비스를 상시 대기(Listen)시키기 위한 데몬(Daemon) 설치 및 설정 필요.
  - 예: `systemctl start vsftpd`
- **Client-side**: 서버에 요청(Request)을 보내기 위한 전용 명령어/소프트웨어 필요.
  - 예: `ftp 192.168.x.x`
- **Dependency**: 
  - 어떤 서비스(NFS 등)는 서버와 클라이언트가 동일한 라이브러리를 공유함.
  - 클라이언트 도구가 기본 OS에 내장된 경우(SSH, Browser)가 많아 별도 설치를 인지 못 할 수 있음.

  ### 🔄 OS Feature Management Comparison

- **Windows**: `Optional Features (Turn Windows features on or off)`
  - 내장된 기능을 체크박스로 활성화/비활성화.
  - GUI 기반이며, 이미 로컬에 바이너리가 준비된 경우가 많음.
- **Linux (Rocky/RHEL)**: `Package Manager (dnf / yum)`
  - 저장소(Repository)에서 패키지를 다운로드하여 설치.
  - CLI 기반이며, 필요한 기능만 골라 설치하므로 시스템이 가벼움.
- **Commonality**: 
  - 특정 프로토콜(Telnet, FTP, NFS)을 사용하려면 서버든 클라이언트든 해당 기능을 명시적으로 '설치' 혹은 '활성화'해야 함.

  ### 📍 Concept: Path (경로)

- **Definition**: 파일 시스템 내에서 특정 파일이나 디렉터리의 위치를 나타내는 문자열.
- **Components of `path = /share`**:
  - `path`: 설정 파일 내에서 대상 디렉터리를 지정하는 키워드.
  - `/`: 리눅스 시스템의 최상위(Root) 지점.
  - `share`: 루트 아래에 위치한 폴더 이름.
- **Key Note**: 
  - 설정 파일에 정의된 `path`는 반드시 실제 스토리지에 존재해야 함.
  - 대소문자를 엄격히 구분함 (`/share` != `/Share`).
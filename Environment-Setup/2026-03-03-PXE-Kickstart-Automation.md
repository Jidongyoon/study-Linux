Automated Linux Installation Server (PXE & Kickstart)
네트워크 부팅을 통해 별도의 USB나 CD 없이 수십 대의 서버에 리눅스(Rocky Linux/CentOS)를 동시에, 자동으로 설치할 수 있는 환경을 구축한 프로젝트입니다.

📖 핵심 개념 (Core Concepts)
1. PXE (Preboot eXecution Environment)
네트워크 인터페이스(NIC)를 통해 컴퓨터를 부팅하는 환경입니다. 클라이언트가 부팅 시 서버로부터 IP와 부팅 파일(Kernel, Initrd)을 받아와 운영체제 설치를 시작하게 합니다.

2. Kickstart (무인 설치 스크립트)
리눅스 설치 시 사용자가 입력해야 하는 설정(언어, 파티션, 비밀번호, 패키지 등)을 미리 작성해둔 텍스트 파일(ks.cfg)입니다. 이 파일을 통해 Zero-Touch 설치가 가능해집니다.

🏗️ 시스템 구성도 (System Architecture)
전체 설치 과정은 아래 4가지 주요 서비스의 협업으로 이루어집니다.

DHCP: 클라이언트에게 IP 주소와 부팅 서버(Next Server), 부팅 파일명을 알려줍니다.

TFTP: 가볍고 빠른 전송 프로토콜로, 부팅 로더(pxelinux.0)와 커널 파일을 전달합니다.

HTTP/FTP: 용량이 큰 OS 설치 이미지(ISO 내용)와 킥스타트 파일을 전달합니다.

Syslinux: 네트워크 부팅을 가능하게 하는 부팅 로더 소프트웨어입니다.

🔄 동작 프로세스 (Workflow)
Client Boot: 클라이언트가 PXE 모드로 부팅하여 네트워크에 DHCP 요청을 보냅니다.

IP & Filename Assign: DHCP 서버가 IP와 함께 pxelinux.0 파일 위치를 응답합니다.

Bootloader Download: 클라이언트가 TFTP를 통해 부팅 로더와 커널(vmlinuz)을 다운로드하고 실행합니다.

Kickstart Fetch: 설치 프로그램(Anaconda)이 구동되면 지정된 URL(FTP/HTTP)에서 ks.cfg 파일을 가져옵니다.

Automated Install: 킥스타트 파일의 설정대로 파티션 설정 및 패키지 설치를 자동으로 진행합니다.

📂 주요 설정 파일 (Key Configurations)
🔹 PXE 부팅 메뉴 (pxelinux.cfg/default)
클라이언트에게 어떤 커널을 부팅하고, 킥스타트 파일은 어디에 있는지 알려주는 이정표입니다.

label linux
  menu label ^Install Rocky Linux 9
  kernel vmlinuz
  append initrd=initrd.img inst.ks=ftp://192.168.111.100/pub/rocky.ks

  킥스타트 파일 (rocky.ks)
시스템 설정을 자동화하는 스크립트입니다.

# 시스템 설정
auth --enableshadow --passalgo=sha512
url --url="ftp://192.168.111.100/pub"
lang ko_KR.UTF-8
timezone Asia/Seoul

# 디스크 자동 파티션
clearpart --all --initlabel
part / --fstype="xfs" --size=10240

# 설치 패키지 선택
%packages
@Server with GUI
%end

기대 효과 (Expected Benefits)
시간 절약: 수동 설치 대비 설치 시간을 80% 이상 단축.

일관성 유지: 모든 서버에 동일한 설정과 패키지 구성을 보장하여 휴먼 에러 방지.

확장성: 대규모 데이터 센터나 클라우드 인프라 구축의 기초 기술로 활용.

📝 구축 시 주의사항 (Note)
Firmware Compatibility: 클라이언트의 가상머신 부팅 모드(Legacy BIOS vs UEFI)에 맞는 부팅 로더 제공 필요.

Network Security: PXE는 네트워크를 통해 데이터를 전송하므로 방화벽(Firewalld) 및 SELinux 설정 유의.


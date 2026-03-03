PXE Server & Kickstart Automated Installation Troubleshooting
이 문서는 CentOS/Rocky Linux 환경에서 PXE(Preboot eXecution Environment) 서버를 구축하고, Kickstart를 이용한 무인 설치 과정에서 발생한 주요 장애 요인과 그 해결 과정을 기록합니다.

📋 시스템 환경 (Environment)
Server OS: Rocky Linux 9 (Virtual Machine)

Services: DHCP, TFTP, HTTP/FTP, Syslinux

Hypervisor: VMware Player

Client: New Virtual Machine (Target for Auto-Installation)

🛠️ 트러블슈팅 요약 (Troubleshooting Summary)

번호,주요 증상 (Issue),원인 분석 (Root Cause),해결 방법 (Resolution)

1,클라이언트가 TFTP 서버에 접속하지 못함,TFTP 서비스가 IPv6에만 바인딩됨,tftp.socket 중지 후 IPv4 서비스 직접 기동

2,네트워크 부팅 시 파일 로드 실패,클라이언트 펌웨어(UEFI)와 부팅 로더(BIOS) 불일치,".vmx 파일을 수정하여 firmware = ""bios"" 강제 설정"

3,Kickstart 구문 분석 오류 (Pane is dead),주석 기호를 별표(*)로 잘못 사용하여 명령어 인식 오류,주석 기호를 표준 샵(#)으로 교체

4,설치 요약 화면에서 그룹 누락 경고,%packages 섹션 내 패키지 그룹 명칭 오타,@Server wiht GUI를 @Server with GUI로 교정

상세 분석 및 해결 (Deep Dive)
1. TFTP IPv4 바인딩 이슈
[증상] netstat 확인 결과, TFTP 포트(69)가 udp6에만 열려 있어 클라이언트의 IPv4 요청을 수신하지 못함.

원인: 시스템 기본값이 IPv6 우선 소켓으로 설정되어 발생.

해결:

systemctl stop tftp.socket
systemctl disable tftp.socket
systemctl restart tftp

VMware Player 펌웨어 호환성 (UEFI vs BIOS)
[증상] 서버 설정이 완벽함에도 클라이언트가 부팅 파일을 읽지 못하고 대기함.

원인: VMware Player의 기본 펌웨어는 UEFI이나, 준비된 pxelinux.0은 Legacy BIOS 전용임. GUI 메뉴에서 변경이 불가능한 Player 버전의 제약.

해결: 가상머신 설정 파일(.vmx)을 메모장으로 열어 수동 수정.

# .vmx 파일 하단에 추가 또는 수정
firmware = "bios"

Kickstart 파일의 구문 오류 (Syntax Error)
[증상] 설치 프로세스 중 Unknown command: *repo 메시지와 함께 Pane is dead 출력 후 중단.

원인: ks.cfg 파일 내에서 설명(Comment)을 위해 사용한 별표(*)를 인스톨러가 명령어로 잘못 해석함.

해결: 모든 비표준 주석 기호를 리눅스 표준인 #으로 변경.

# AS-IS (Error)
*repo --name="AppStream"

# TO-BE (Fixed)
# repo --name="AppStream"

. 패키지 그룹 명칭 오타 (Typo in Packages Section)
[증상] missing groups or modules: @Server wiht GUI 경고창 발생.

원인: 영문 스펠링 with를 wiht로 잘못 입력하여 해당 패키지 그룹을 리포지토리에서 찾지 못함.

해결: rocky.ks 파일의 %packages 섹션 오타 수정.

%packages
@Server with GUI  # Corrected spelling
%end

최종 교훈 (Lessons Learned)
정확한 경로 및 스펠링: 자동화 스크립트에서 글자 하나(s, *, wiht)는 전체 시스템을 마비시키는 치명적인 오류가 됨.

펌웨어 매칭: PXE 환경 구축 시 서버가 제공하는 로더와 클라이언트의 부팅 모드(BIOS/UEFI)를 반드시 일치시켜야 함.

무료 툴의 제약 극복: VMware Player처럼 GUI 설정을 제한하는 경우 설정 파일(.vmx) 수동 편집 등 엔지니어링 접근 방식이 유효함.

✅ Result
모든 트러블슈팅 완료 후, 클라이언트는 서버로부터 IP를 할당받아 커널을 로드하고, Kickstart를 통해 사용자 개입 없이 OS 설치를 성공적으로 완료함.


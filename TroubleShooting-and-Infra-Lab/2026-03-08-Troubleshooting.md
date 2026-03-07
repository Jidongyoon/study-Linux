[Troubleshooting] Rocky Linux 9 패키지 매니저(dnf) 및 DNS 네임 서버 장애 복구
1. 개요
Rocky Linux 9 환경에서 **FTP 서버(vsftpd)**와 **DNS 서버(bind)**를 구축하는 과정에서 발생한 패키지 설치 실패 및 서비스 기동 오류에 대한 원인 분석과 해결 과정을 기록합니다.

2. 장애 상황 1: 패키지 설치 실패 (dnf install vsftpd)
[현상]
dnf install 실행 시 Could not resolve host 에러 발생.

커스텀 저장소 설정 후 repomd.xml parser error 발생하며 메타데이터 다운로드 실패.

[원인 분석]
DNS 설정 부재: 가상머신이 외부 도메인(한빛미디어, 카카오 등)의 IP를 찾지 못함.

Repo 파일 문법 오류: This.repo 파일 작성 시 cat <<EOF 같은 쉘 명령어를 파일 내용 내부에 직접 기재하여 파싱 에러 발생.

[해결 방법]
네트워크 식별: /etc/resolv.conf에 구글 DNS(8.8.8.8)를 추가하여 도메인 해석 권한 부여.

저장소 초기화: 공식 Rocky Linux 미러 서버 주소로 This.repo 재작성

cat <<EOF > /etc/yum.repos.d/This.repo
[baseos]
name=Rocky Linux 9 - BaseOS
baseurl=https://dl.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/
gpgcheck=0
EOF

캐시 정리: dnf clean all 실행 후 재설치 성공.

3. 장애 상황 2: DNS 서비스 기동 실패 (named.service)
[현상]
네임 서버 설정 후 systemctl restart named 실행 시 Job for named.service failed 에러 발생.

[원인 분석]
문법 오류 (Syntax Error): /etc/named.conf 설정 파일 내부에 세미콜론(;) 누락 및 중괄호 {} 닫기 미흡.

[해결 방법]
설정 검사 도구 활용: named-checkconf /etc/named.conf 명령어로 에러 발생 라인 특정.

문법 교정:

listen-on port 53 { any; }; 끝에 세미콜론 추가.

allow-query { any; }; 설정으로 외부 클라이언트 허용.

서비스 재시작: 문법 수정 후 정상 기동 확인.

4. 장애 상황 3: 클라이언트 접속 불가
[현상]
서버 가상머신에서는 접속이 되나, 외부 클라이언트(Windows/Client VM)에서 도메인 접속 불가.

[원인 분석]
클라이언트 DNS 미지정: 클라이언트가 사설 DNS 서버를 바라보지 않음.

방화벽 차단: 서버의 53번(DNS), 80번(HTTP) 포트가 닫혀 있음.

[해결 방법]
포트 개방:

firewall-cmd --permanent --add-service=dns
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

클라이언트 설정: 네트워크 설정에서 DNS 서버 주소를 서버 IP(192.168.111.100)로 수동 지정.

5. 요약 및 교훈
순정의 힘: 복잡한 커스텀 설정보다 기본(Default) 저장소 설정이 가장 안정적일 수 있음.

점 하나, 땀 하나: 리눅스 설정 파일에서 ;와 .은 생명과도 같음. 수정 후에는 반드시 check 명령어로 검수할 것.

계층적 접근: 네트워크 연결(Ping) -> 이름 해석(DNS) -> 서비스(HTTP/FTP) 순으로 단계별 점검이 필수적임.

# [Troubleshooting] Rocky Linux 9 패키지 매니저(dnf) 및 SSH 서비스 장애 복구

## 1. 개요 (Overview)
- **대상 시스템**: Rocky Linux 9 (Virtual Machine)
- **장애 유형**: 외부 네트워크 통신 및 dnf 패키지 업데이트 불가, SSH 접속 불능
- **발생 원인**: 레포지토리(Repository) 주소 설정 오류 및 의존성 라이브러리 충돌

## 2. 장애 상황 (Issue Details)
- `dnf update` 실행 시 `404 Not Found` 에러 발생과 함께 메타데이터 동기화 실패.
- 네트워크 인터페이스 활성화 및 DNS 설정 누락으로 외부 도메인 통신 불가.
- `sshd` 서비스 기동 시 `OpenSSL` 라이브러리 버전 불일치로 인한 Connection Refused 발생.

## 3. 원인 분석 (Root Cause Analysis)
- **잘못된 경로 참조**: `/etc/yum.repos.d/This.repo` 파일 내 `baseurl` 경로에 불필요한 `/vault/` 경로가 삽입됨.
- **버전 불일치**: Rocky Linux 9은 최신 릴리스로, 아카이브용인 `vault`가 아닌 공식 배포 경로인 `pub`를 참조해야 함.
- **설정 혼선**: 반복적인 `sed` 명령어 사용으로 인해 주소 체계 내에 중복된 경로명이 삽입되어 구문 오류 유발.

## 4. 해결 과정 (Resolution Steps)

### 4.1 네트워크 활성화 및 DNS 설정
통신 채널 확보를 위해 이더넷 인터페이스를 연결하고 구글 퍼블릭 DNS를 수동 등록함.
```bash
nmcli device connect ens160
echo "nameserver 8.8.8.8" > /etc/resolv.conf

4.2 레포지토리 파일 재생성 (Overwrite)
설정이 꼬였을 때는 부분 수정보다 정상 주소로 파일을 완전히 새로 작성하는 것이 가장 확실합니다.

cat <<EOF > /etc/yum.repos.d/This.repo
[baseos]
name=Rocky Linux 9 - BaseOS
baseurl=https://dl.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/https://dl.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/
gpgcheck=0

[appstream]
name=Rocky Linux 9 - AppStream
baseurl=https://dl.rockylinux.org/pub/rocky/9/AppStream/x86_64/os/https://dl.rockylinux.org/pub/rocky/9/AppStream/x86_64/os/
gpgcheck=0
EOF

[3단계] 캐시 정리 및 라이브러리 업데이트
잘못된 메타데이터를 삭제하고 시스템을 최신 상태로 갱신합니다.

dnf clean all
dnf update -y openssl
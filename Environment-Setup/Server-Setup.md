# 🐧 Rocky Linux 9 서버 구축 및 초기 설정 기록

VMware Workstation 17.6.4 버전을 사용하여 실습용 메인 서버(Server) 구축을 완료했습니다.

## 1. 가상머신 세부 사양
- **이름:** Server
- **OS:** Rocky Linux 9.x (64-bit)
- **CPU:** 1 Processor / 1 Core
- **RAM:** 4GB
- **Disk:** 80GB (NVMe)
- **Network:** NAT (외부 인터넷 통신 가능)

## 2. 주요 설정 사항
- **파티션 설정:** 강좌 가이드에 따라 `/`, `/boot`, `/home`, `swap` 등 수동 분할 완료
- **네트워크:** 설치 단계에서 이더넷(ens160) 활성화 및 호스트 이름 설정 완료
- **사용자 설정:** root 계정 활성화 및 관리자 권한을 가진 일반 사용자 계정 생성 완료

## 3. 설치 후 확인
- 부팅 후 정상 로그인 확인
- `ping google.com`을 통한 외부 네트워크 통신 확인 (NAT 작동 확인)
# 📅 가상 머신 시스템 설정 기록 (2026-01-31)
## Server(B) 및 클라이언트 환경 구성 완료

### 1. Server(B) [Linux 서버 - AlmaLinux 9]
- **Hostname**: `server-b`
- **관리 도구**: `nano` 편집기를 활용한 시스템 설정 파일 제어 환경 구축.
- **패키지 환경**: 전용 레포지토리(`This.repo`) 설정을 통한 패키지 관리 체계 확립.
- **네트워크**: 서버 운영을 위한 인터페이스 활성화 및 초기화 완료.

### 2. Linux Client [클라이언트 - AlmaLinux 9]
- **네트워크**: **자동 IP 할당 (DHCP)** 모드로 동작.
- **사용자 환경**: GUI(Workstation) 모드 설치 및 실습용 일반 사용자 계정 생성.

### 3. WinClient [Windows 클라이언트 - Windows 11 Enterprise]
- **네트워크**: **자동 IP 할당 (DHCP)** 모드로 동작.
- **시스템 사양**: Processor(2 Cores), RAM(4GB) 할당 (Windows 11 최적화 사양).
- **최적화**: VMware Tools 설치 완료 및 실습용 로컬 관리자 계정 구성.

---
### 📝 실습 메모
- Server(B)는 고정적인 서비스 제공을 위한 수동 설정 환경으로, 클라이언트들은 유연한 접속을 위한 DHCP 환경으로 이원화하여 구축함.
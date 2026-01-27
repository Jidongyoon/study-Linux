# 🛠️ VMware Workstation 가상머신 구축 상세 기록

본 문서는 클라우드 엔지니어링 실습 환경을 위해 구축한 가상머신 4대의 상세 설정 값을 기록합니다.

## 1. 가상머신 리스트 및 사양 (Summary)

| 머신 이름 | 설치 ISO | 하드 용량 | 메모리(RAM) | 네트워크 |
| :--- | :--- | :--- | :--- | :--- |
| **Server** | Rocky Linux 9 | 80GB | 2GB | NAT |
| **Server(B)** | Rocky Linux 9 | 40GB | 512MB | NAT |
| **Client** | Rocky Linux 9 | 40GB | 2GB | NAT |
| **WinClient** | Windows 10 | 60GB | 1GB | NAT |

## 2. 상세 설정 정보
- **Hypervisor:** VMware Workstation 17 Player
- **Guest OS:** Rocky Linux 9 (64-bit) 및 Windows 10
- **Network Type:** 외부 인터넷 통신을 위해 모든 머신에 **NAT** 방식 적용
- **특이사항:** - `Server(B)`는 텍스트 모드로 설치하여 자원 사용량을 최소화함.
  - `WinClient`는 평가판(64-bit)으로 환경 구축 완료.

## 3. 구축 확인 스크린샷
*(여기에 실습 화면 스크린샷을 찍어서 넣으면 더 완벽한 기록이 됩니다!)*
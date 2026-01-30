# 🛠️ 종합 트러블슈팅 로그 (2026-01-31)

## [Issue 01] Server(B) 패키지 저장소(Repo) 수동 구성
- **현상**: 기본 Mirror 사이트 접근 실패로 인해 `dnf install` 등 패키지 설치 불가.
- **해결**: 
  1. `sudo nano /etc/yum.repos.d/This.repo` 명령어로 설정 파일 생성.
  2. `[ThisRepo]` 섹션에 유효한 `baseurl`, `enabled=1`, `gpgcheck=0` 정보를 직접 타이핑하여 입력.
  3. 저장 후 패키지 매니저 정상 작동 확인.

## [Issue 02] Windows 11 하드웨어 요구 사항(TPM/RAM/CPU) 우회
- **현상**: 가상 머신 사양 미달로 "이 PC에서는 Windows 11을 실행할 수 없습니다" 메시지 발생.
- **해결**: `Shift + F10` → `regedit` 실행 후 `HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig` 경로에 `BypassTPMCheck`, `BypassRAMCheck`, `BypassCPUCheck` 값을 각각 1(DWORD)로 설정하여 설치 진행.

## [Issue 03] Windows 10 ISO 다운로드 경로 차단 이슈
- **현상**: MS 정책으로 인해 Windows 10 다운로드 페이지가 지원 종료 안내 페이지로 자동 리다이렉트됨.
- **해결**: 실무 대응력 확보를 위해 설치 대상을 최신 버전인 Windows 11 Enterprise로 변경하여 환경 구축 완료.
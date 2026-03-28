# 🐧 Linux Package Manager & Troubleshooting (CentOS vs Ubuntu)

이 문서는 클라우드 엔지니어 실무 과정에서 학습한 **패키지 매니저의 구조, 보안(GPG), 그리고 실무 트러블슈팅 사례**를 정리한 기술 문서입니다.

---

## 1. 패키지 인덱스 캐시 데이터 관리 (`/var/lib/apt/lists/`)

패키지 매니저는 실제 파일을 받기 전, 서버로부터 패키지 목록과 지문(Checksum)이 담긴 '인덱스 파일'을 먼저 가져와 로컬에 저장합니다.

### 🔍 문제 상황: 인덱스 데이터 손상 (Hash Sum Mismatch)
* **원인**: 로컬 인덱스 파일이 네트워크 오류나 강제 종료로 인해 실제 서버 데이터와 일치하지 않을 때 발생.
* **증상**: `sudo apt update` 시 `W: Failed to fetch...` 또는 `Hash Sum Mismatch` 경고 발생.
* **해결 방법 (강제 초기화)**:
  ```bash
  # 1. 기존 오염된 인덱스 파일 전량 삭제
  $ sudo rm -rf /var/lib/apt/lists/*

  # 2. 패키지 목록 새로고침 (서버에서 최신 데이터 다시 다운로드)
  $ sudo apt update

  2. 외부 저장소 등록 및 문법 오류 (Malformed entry)
/etc/apt/sources.list 및 .list 파일은 엄격한 문법을 따릅니다. 형식이 틀리면 전체 패키지 시스템이 중단됩니다.

🔍 문제 상황: 문법 오류 발생
에러 메시지: E: Malformed entry [Line No] in list file ... (Component)

원인: 주소 형식([유형] [URL] [코드네임] [카테고리]) 중 일부가 누락되거나 띄어쓰기 오타가 난 경우.

트러블슈팅 방법:

vim 실행 후 :set nu 명령어로 에러 라인 번호 확인.

해당 라인의 OS 코드네임(예: jammy) 및 대괄호([]) 옵션 문법 체크.

불완전하거나 중복된 줄은 삭제(dd) 처리.

3. GPG(GNU Privacy Guard) 보안 및 신뢰 구축
외부 저장소(Docker 등)를 추가할 때는 패키지 변조 방지를 위해 '공개 키(Public Key)' 등록이 필수입니다.

🛡️ GPG 키 추가 및 검증 프로세스
키 다운로드 및 이진화:

$ curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

권한 수정: 시스템(apt)이 읽을 수 있게 권한 부여.

$ sudo chmod a+r /etc/apt/keyrings/docker.gpg

파일 정체 확인 (file 명령어):

$ file /etc/apt/keyrings/docker.gpg
# 출력: OpenPGP Public Key (정상적인 보안 키 파일임을 확인)

4. 모듈형 저장소 관리 (Best Practice)
메인 설정 파일(sources.list)을 직접 수정하기보다 서비스별 전용 파일을 생성하여 관리하는 것이 실무 표준입니다.

권장 경로: /etc/apt/sources.list.d/docker.list

설정 예시:
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) jammy stable

이유: 메인 설정 파일의 오염을 방지하고, 특정 서비스 삭제 시 해당 파일만 지우면 되므로 유지보수가 편리함.

구분,CentOS (RHEL),Ubuntu (Debian)
캐시 초기화,yum clean all,rm -rf /var/lib/apt/lists/*
저장소 경로,/etc/yum.repos.d/*.repo,/etc/apt/sources.list.d/*.list
버전 표기,$releasever (변수),"jammy, focal (코드네임)"
파일 분석,file [파일명],file [파일명]

💡 엔지니어의 한마디: "작동하게 만드는 것은 기본이다. 관리가 용이하고 보안이 검증된 설정을 유지하는 것이 진짜 실력이다."

Learning Note: 2026-03-29 실습 완료 (Cloud Engineer Roadmap)





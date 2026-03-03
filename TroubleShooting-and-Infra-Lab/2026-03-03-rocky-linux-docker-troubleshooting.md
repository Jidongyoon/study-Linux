[Troubleshooting] Rocky Linux 9 Docker 설치 및 의존성 오류 해결 환경 구축
1. 개요
일시: 2026-03-03

환경: VMware Workstation 17 Player, Rocky Linux 9.x (Blue Onyx)

주요 내용: Docker 엔진 설치 중 발생하는 libnftables.so.1 의존성 오류 해결 및 가상 머신 내 브라우저 렌더링 이슈 대응

2. 문제 상황 (Issue)
2.1. Docker 설치 중 의존성 오류
dnf install 명령어를 통해 도커를 설치하려 했으나, 다음과 같은 오류 메시지와 함께 설치가 중단됨.

에러 메시지: Error: Problem: cannot install the best candidate for the job - package docker-ce-... requires libnftables.so.1(LIBNFTABLES_1)(64bit), but none of the providers can be installed

상황: dnf update를 실행했음에도 불구하고 특정 라이브러리(libnftables)를 찾지 못하거나 버전이 맞지 않는 현상 발생.

2.2. 브라우저(Firefox) 화면 출력 오류
Docker Hub 사이트 접속 시, 초기 로딩 화면이나 쿠키 동의 배너는 출력되나 이후 화면이 백색(White-out)으로 변하며 멈추는 현상 발생.

3. 원인 파악 (Root Cause Analysis)
3.1. 라이브러리 버전 불일치 및 경로 인식 문제
Rocky Linux 9은 최신 nftables를 사용하지만, Docker의 공식 CentOS/RHEL용 레포지토리는 특정 버전의 libnftables.so.1을 고집하는 경향이 있음.

find 명령어 확인 결과, 시스템 내에 유사한 라이브러리 파일은 존재하나 패키지 매니저가 이를 도커 설치 조건과 매칭시키지 못함.

3.2. 가상 머신 그래픽 가속 충돌
VMware 환경에서 리눅스 GUI 브라우저를 실행할 때, '3D Graphics Acceleration' 기능이 브라우저의 하드웨어 가속과 충돌하여 렌더링 오류를 일으킴.

4. 해결 과정 (Resolution Steps)
단계 1: 시스템 라이브러리 상태 확인
먼저 시스템 내에 문제가 되는 라이브러리가 있는지 확인했습니다.
find /usr/lib64 -name "libnftables.so*"

결과: /usr/lib64/libnftables.so.1.0.0 등의 파일이 존재함을 확인.

단계 2: 의존성 무시 및 최적 후보 설치 (핵심 해결책)
단순한 설치 명령어가 아닌, 충돌하는 패키지를 교체 허용하고 최적의 후보를 찾는 옵션을 추가하여 설치를 강제했습니다.

dnf install -y docker-ce docker-ce-cli containerd.io --allowerasing --nobest

--allowerasing: 충돌하는 기존 패키지 삭제/교체 허용

--nobest: 의존성 문제가 없는 차선책 버전 설치 허용

단계 3: 도커 서비스 활성화 및 검증
설치 완료 후 서비스를 시작하고 정상 작동 여부를 확인했습니다.

systemctl enable --now docker
docker run hello-world

단계 4: 컨테이너 내부 진입 (Bash Shell 실행)
강의 내용에 따라 우분투 이미지를 활용해 컨테이너 내부 배쉬 셸에 접속 성공했습니다.

docker run -it ubuntu bash
# 컨테이너 내부 접속 확인
cat /etc/issue

5. 결과 및 배운 점
5.1. 결과
Docker 서비스 상태 active (running) 확인.

우분투 컨테이너 내부 진입 및 리눅스 명령어 실행 확인.

5.2. 배운 점
실무형 트러블슈팅: 에러 메시지에 명시된 라이브러리 파일명을 직접 검색하고, 패키지 매니저의 고급 옵션(--allowerasing)을 활용하는 법을 익힘.

우회 전략: 가상 머신 내부의 브라우저가 불안정할 경우, 호스트 PC(윈도우) 브라우저를 활용하고 터미널로 명령어를 전달하는 효율적인 작업 흐름을 파악함.

가상화의 이해: 호스트 OS(Windows) -> 가상 OS(Rocky Linux) -> 컨테이너(Ubuntu)로 이어지는 계층 구조를 실습을 통해 체득함.

6. 참고 명령어 모음
서비스 상태 확인: systemctl status docker

컨테이너 탈출: exit 또는 Ctrl+P, Q

강제 링크 생성(필요시): ln -s [원본파일] [심볼릭링크]



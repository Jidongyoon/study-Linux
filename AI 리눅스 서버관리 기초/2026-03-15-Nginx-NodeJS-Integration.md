Nginx Reverse Proxy 배포 및 네트워크 분석 리포트
본 문서는 오픈소스 프로젝트의 클론부터 Nginx 리버스 프록시 연동, 그리고 실시간 트래픽 분석을 통한 시스템 검증 과정을 기록한 엔지니어링 로그입니다.

1. 애플리케이션 환경 구축 (App Setup)

1.1 소스코드 확보 (Git Clone)
가장 먼저 깃허브 오픈소스 저장소로부터 프로젝트를 로컬 서버로 복제함.

git clone [저장소_URL]
cd [프로젝트_명]

1.2 의존성 설치 및 앱 구동
Node.js 환경에서 필요한 패키지를 설치하고 백엔드 서버를 3000번 포트에서 활성화함.

npm install
npm start  # 또는 node app.js

검증: netstat -nltp | grep 3000 명령어를 통해 해당 포트가 LISTEN 상태임을 확인.

2. Nginx 리버스 프록시 설정 (Reverse Proxy)
2.1 프록시 서버 설계
클라이언트가 80번(기본 HTTP) 포트로 접속했을 때, 내부의 3000번 포트로 요청을 중계하도록 nginx.conf를 수정함.

# /etc/nginx/nginx.conf
location / {
    proxy_pass http://localhost:3000;
   }

2.2 서비스 제어 및 반영
전통적인 시스템 서비스 관리 명령어를 사용하여 설정을 적용함.

sudo nginx -t             # 설정 파일 문법 검사
sudo service nginx restart # Nginx 서비스 재시작 및 설정 반영

3. 네트워크 트러블슈팅 (Troubleshooting)
3.1 호스트-게스트 통신 이슈
현상: 리눅스 내부에서는 curl localhost가 성공하나, 윈도우 브라우저에서는 접속 불가.

원인: NAT 환경의 사설 IP(10.0.2.15)는 외부(Host)에서 직접 접근이 불가능한 구조임.

해결:

가상머신 네트워크를 **어댑터 브릿지(Bridged Adapter)**로 변경.

공유기로부터 192.168.x.x 대역의 새로운 독립 IP 할당 확인.

필요 시 리눅스 방화벽 해제 (sudo systemctl stop firewalld).

4. 실시간 트래픽 분석 (Traffic Analysis)
4.1 iftop을 이용한 데이터 흐름 시각화
루프백 인터페이스(lo)를 모니터링하여 Nginx와 Node.js 간의 통신을 물리적으로 확인.

명령어: sudo iftop -i lo

관측: 브라우저 요청 시 localhost 간의 TX(송신) 데이터 수치가 즉각적으로 상승함. 루프백 특성상 송신이 곧 수신이므로 시스템 내부 연동이 완벽함을 증명함.

4.2 watch 명령어를 통한 자동화 테스트
반복적인 요청을 자동화하여 서버의 안정성을 테스트함.

명령어
watch -n 1 curl -s localhost

분석: 1초 단위의 자동 요청을 통해 서버 로그 및 트래픽 변화를 실시간 관측함. 이는 DoS(서비스 거부 공격)의 기초 메커니즘을 이해하고 시스템 가용성을 검증하는 과정임.

5. 결론 및 성과 (Conclusion)
Full-Stack Deployment: 소스코드 확보부터 웹 서버 연동까지의 전체 배포 사이클을 완주함.

Infrastructure Insight: 리버스 프록시를 통해 포트 간의 경계를 허물고, 네트워크 대역(NAT vs Bridged)에 따른 통신 원리를 터득함.

Real-time Monitoring: 로그와 트래픽 데이터를 바탕으로 추상적인 서버 동작을 수치로 시각화함.





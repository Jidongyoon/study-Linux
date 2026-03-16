Full-Stack Infrastructure Setup & Performance Testing
본 실습은 웹 서버와 WAS의 유기적인 연동(Reverse Proxy)을 이해하고, 실제 트래픽 부하 상황에서 시스템이 어떻게 반응하는지 모니터링 및 분석하는 것을 목적으로 합니다.

📅 실습 일시: 2026-03-16 ~ 17
🏗️ 1. Infrastructure Architecture
구축한 시스템의 전체 흐름은 다음과 같습니다.

Web Server: Nginx / Apache (Port 80)

WAS (Web Application Server): Gunicorn + Django (Port 8000)

Database: SQLite (Default Django DB)

Monitoring Tools: iptraf-ng, bmon, ab (Apache Bench)

🛠️ 2. Step-by-Step Implementation
2-1. Django WAS 환경 구축
Access Control: 외부 접속 허용을 위해 settings.py 수정.

ALLOWED_HOSTS = ['*'] 설정 반영.

Gunicorn 연동: 실전 배포를 위해 runserver 대신 WSGI 서버인 Gunicorn 활용.

gunicorn --config gunicorn-cfg.py core.wsgi

2-2. Web Server (Reverse Proxy) 설정
Nginx & Apache: 80번 포트로 들어오는 요청을 8000번(Django)으로 전달하는 리버스 프록시 구성.

Port Forwarding: 사용자에게 포트 번호(8000)를 노출하지 않고 80번 포트만으로 서비스 접근 가능하도록 구현.

🧪 3. Network Stress Test & Monitoring
3-1. Traffic Monitoring
iptraf-ng: 패킷 단위의 상세 네트워크 트래픽 감시.

bmon: 대역폭 사용량 실시간 시각화.

3-2. Load Testing (Apache Bench)
서버의 한계를 측정하기 위해 다양한 시나리오로 부하 테스트 진행.

명령어: ab -n [총 요청수] -c [동시 요청수] [URL]

시나리오:

웹 서버 경유 테스트: http://localhost/login/ (Reverse Proxy 성능 확인)

WAS 직접 테스트: http://localhost:8000/login/ (순수 WAS 성능 확인)

### ⚠️ 4. Troubleshooting Ledger (문제 해결 기록)

| 이슈 상황 | 원인 분석 | 해결 방법 |
| :--- | :--- | :--- |
| **Permission Denied** | 관리자 권한 없이 시스템 파일 수정 시도 | 명령어 앞에 `sudo`를 붙여 권한 부여 후 실행 |
| **Window Resize Error** | `iptraf-ng` 실행 시 터미널 창 크기가 너무 작음 | 터미널 창을 가로 80, 세로 24 이상으로 확대 후 재실행 |
| **Address already in use** | 이전 프로세스(`runserver`)가 8000번 포트를 점유 중 | `sudo fuser -k 8000/tcp` 명령어로 기존 프로세스 강제 종료 |
| **Unit cannot be reloaded** | Nginx가 중지(`dead`)된 상태에서 설정 재로드 시도 | `sudo systemctl start nginx`로 서비스 먼저 기동 |
| **Old Page Displayed** | 브라우저 캐시 또는 이전 실습(Apache) 프로세스 간섭 | `sudo systemctl stop httpd` 실행 및 브라우저 강력 새로고침(`Ctrl+Shift+R`) |
| **Hardware Noise (Fan)** | 과도한 부하 테스트로 인한 CPU 온도 급상승 및 팬 소음 | `sudo pkill -9 hping3` 명령어로 부하 유발 프로세스 즉시 종료 |



5. Final Insights
Reverse Proxy의 중요성: 웹 서버를 앞단에 두어 보안을 강화하고, 효율적으로 트래픽을 관리하는 아키텍처의 이점을 이해함.

Resource Management: 소프트웨어의 과도한 부하(flood test)가 하드웨어(CPU 열, 팬 소음)에 미치는 물리적 영향을 실시간 관찰함.

Monitoring: iptraf-ng와 bmon을 통해 보이지 않는 패킷의 흐름을 시각화하여 시스템 상태를 진단하는 능력을 배양함.

Master's Final Log:

"단순한 코드 작성을 넘어 서버와 서버가 대화하는 법을 배웠다. 이제 어떤 서비스든 올릴 수 있는 탄탄한 인프라 기초 체력을 갖추게 되었다."


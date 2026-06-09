# 📝 [TIL] 2026-06-09 | Rocky Linux 10 인프라 구축 및 웹 서버 최적화 종합 실습

오늘 Rocky Linux 10 환경에서 Nginx, FastAPI, Redis, MariaDB를 연동한 쇼핑몰 인프라 구조를 배우고, 리눅스 커널의 캐시 시스템 원리와 Nginx 워커 프로세스 최적화 테스트를 종합적으로 학습했습니다.

---

## 1. 양말 쇼핑몰 웹 서버 기본 인프라 구조
- **전체 흐름:** `유저 요청 ➡️ Nginx (80번 포트) ➡️ 포트 포워딩 (80:8000) ➡️ FastAPI ➡️ Redis (캐시) ➡️ MariaDB (DB)`
- **Nginx 프록시 설정 경로:** `/etc/nginx/conf.d/default.conf`
- **핵심 설정 코드:**
  ```nginx
  server {
      listen       80;
      server_name  localhost;

      location / {
          proxy_pass [http://127.0.0.1:8000](http://127.0.0.1:8000); # 맨 뒤에 슬래시(/)를 빼야 경로 배달 사고가 안 남
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }

  Rocky Linux 10 필수 방화벽 및 보안 정책 해제:

  # Nginx의 네트워크 중계 허용 (502 Bad Gateway 에러 방지)
sudo setsebool -P httpd_can_network_connect 1

# 80번 포트 방화벽 오픈
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --reload

2. 엔지니어의 디버깅 원칙: "오류는 결과이자 현상일 뿐이다"
에러(예: 404 Not Found, 502 Bad Gateway)가 발생했을 때 섣부르게 코드를 고치기 전에 데이터가 이동하는 길목의 가장 당연한 단계부터 하나씩 격리하며 추적해야 합니다.

🔍 시스템 레벨 추적 3단계 규칙
실행 확인: nginx, fastapi(uvicorn) 프로그램이 실제로 백그라운드에 정상 구동 중인지 확인 (ps -ef, systemctl status)

설정 검사: Nginx 설정 파일에 문법적 오타나 타이포가 없는지 사전 검수 (sudo nginx -t)

설정 우선순위 파악: 하위 폴더별 설정(conf.d/*.conf)이 마스터 통제실인 메인 설정(nginx.conf)의 시스템 범위 설정에 밀려 무시(덮어쓰기)당하고 있진 않은지 확인

3. 리눅스 메모리 관리와 캐시 시스템의 이해 (main_redis.py 실습)
리눅스 커널이 디스크 I/O 병목을 줄이기 위해 사용하는 캐시 메커니즘과 메모리 분할 구조를 확인했습니다.

💡 리눅스 메모리 조각 단위 (페이지: 4KB) 구조
Anonymous 용도: 스택(Stack), 힙(Heap) 등 프로세스가 동작하기 위해 순수하게 사용하는 실시간 메모리 영역

Pagecache 용도: 디스크 블록의 데이터를 RAM에 임시로 유지하여 읽기 성능을 극대화하는 용도

📊 메모리 강제 삭제(drop_caches) 성능 비교 측정 결과
캐시 비우기 명령: echo 3 > /proc/sys/vm/drop_caches 실행 후 전량 초기화 상태에서 속도 변화 테스트

실전 측정 속도 추이:

첫 번째 요청 (0.3s): 캐시가 없어 디스크나 DB에서 직접 읽어오는 본래의 속도 (가장 느림)

두 번째 요청 (0.02s): 리눅스 Pagecache에 데이터가 적재되어 즉시 반환 (초고속)

캐시 비우기 후 세 번째 요청 (0.1s): Pagecache가 강제 삭제되어 속도가 다시 떨어짐

네 번째 요청 (0.02s): 다시 Pagecache 메모리에 캐싱 완료되어 고속 속도 회복

❓ Redis 내부 캐시 확인 및 제어 명령어
내부 캐시 존재 여부 확인: Redis CLI 환경에 접속하여 키 구조 검색 (keys *)

내부 캐시 비우기: FLUSHALL 또는 FLUSHDB 명령어를 통해 처리하여 메모리 완전 초기화

⏱️ 시간 비교 함수 차이점
time 명령어: 프로그램이 시작해서 완전히 끝날 때까지 걸린 전체 시스템/유저 레벨의 절대적인 벽시계 시간(Wall-clock time) 측정

time.perf_counter(): 파이썬 코드 내부에서 아주 미세한 코드 실행 구간의 하드웨어 클록 레퍼런스 기준 고정밀 시간 간격 측정

4. 실시간 서버 모니터링 및 Nginx 성능 최적화 실습
서버의 교통량과 일꾼들의 피로도를 실시간으로 모니터링하며 Nginx 설정을 튜닝하는 실습을 진행했습니다.

🛠️ 핵심 모니터링 도구 역할
bmon (Bandwidth Monitor): 외부 유저 인입 시 네트워크 카드 인터페이스로 데이터 패킷이 정상 도달하는지 실시간 대역폭 트래픽 모니터링

htop: CPU 코어별 사용량, RAM 메모리 잔여량, 실행 중인 프로세스 자원 점유율을 컬러 막대그래프로 실시간 시각화 추적

🧑‍🍳 Nginx 워커 프로세스(worker_processes) 튜닝 테스트
메인 설정 파일 (/etc/nginx/nginx.conf)에서 요청을 직접 처리하는 독립된 일꾼의 수인 worker_processes 설정을 변경해가며 부하 테스트를 검증했습니다.

worker_processes 1; 세팅: 대량 트래픽(httpd-tools 패키지의 ab 부하 테스트) 유입 시 특정 CPU 단일 코어 하나만 100%를 찍으며 병목 현상 발생.

worker_processes 2; (또는 auto) 세팅: 다중 CPU 코어 환경에서 개별 워커 프로세스들이 일을 사이좋게 분산 처리하여 초당 요청 처리 속도가 대폭 향상됨을 계기판으로 확인.

프로세스 수 확인 명령어: ps -ef | grep nginx를 통해 마스터 프로세스 아래 구동 중인 일꾼 개수 매칭 확인.

🎯 오늘의 한 줄 요약
에러가 났을 때 당황해서 여기저기 쑤시지 말고, 데이터가 이동하는 인프라 구조를 바탕으로 프로그램 실행 상태부터 메인 시스템 설정 권한까지 단계별로 그물망을 좁혀가자!
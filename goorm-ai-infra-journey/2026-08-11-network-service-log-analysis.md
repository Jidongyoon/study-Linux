# 🌐 네트워크 · 서비스 · 로그 장애 분석 실습

> 2026-08-11 학습 기록

## 1. 네트워크 통신의 기본 구조

서버 간 통신에서는 IP 주소와 Port(포트)를 통해
특정 서비스에 연결한다.

IP 주소는 어떤 서버인지 식별하고,
Port는 해당 서버에서 어떤 서비스와 통신할 것인지 구분한다.

예시:

54.xxx.xxx.xxx:80

- IP 주소: 54.xxx.xxx.xxx
- Port: 80
- 80번 Port는 일반적으로 HTTP 서비스에서 사용된다.

따라서 네트워크 장애가 발생했을 때는
IP 주소뿐만 아니라 Port까지 함께 확인해야 한다.

---

## 2. Binding과 Listening

서비스가 특정 IP와 Port에서 연결을 받을 수 있도록
소켓을 연결하는 것을 Binding이라고 한다.

서버에서 어떤 Port가 Listening 상태인지 확인:

    ss -tlnp

특정 Port 확인:

    ss -tlnp | grep 8002

결과가 없다면 해당 Port에서
현재 Listening 중인 서비스가 없다는 의미다.

네트워크 문제를 확인할 때는
먼저 서비스가 실행되고 있는지,
그리고 실제로 원하는 Port에서 Listening 중인지 확인해야 한다.

---

## 3. Port 연결 테스트

서버 내부에서는 ss를 이용해
현재 Listening 중인 Port를 확인할 수 있다.

외부에서 특정 Port까지 실제 연결이 가능한지는
nc를 이용해 확인할 수 있다.

    nc -zv <IP주소> <Port>

예:

    nc -zv 54.xxx.xxx.xxx 80

네트워크 장애가 발생하면 다음 순서로 범위를 좁혀간다.

서비스 실행 여부
→ Port Listening 여부
→ 서버 방화벽
→ 클라우드 보안 그룹
→ 외부 연결 여부

---

## 4. 방화벽과 Port

서비스가 정상적으로 실행되고 있어도
방화벽이 해당 Port를 차단하면 외부에서는 접속할 수 없다.

UFW 상태 확인:

    sudo ufw status

Port 허용:

    sudo ufw allow 8002/tcp

이미 동일한 규칙이 존재하면
Skipping adding existing rule 메시지가 출력될 수 있다.

이는 오류가 아니라
해당 방화벽 규칙이 이미 존재한다는 의미다.

---

## 5. 서비스 상태 확인

서비스 상태 확인:

    systemctl status <서비스명>

예:

    systemctl status ssh

서비스가 Active 상태라고 해서
모든 문제가 해결되는 것은 아니다.

서비스 상태와 함께
프로세스와 로그를 확인해야 한다.

프로세스 확인:

    ps aux

실시간 프로세스 확인:

    top

서비스 로그 확인:

    journalctl -u <서비스명>

즉,

프로세스
→ 서비스
→ 로그

를 연결해서 확인하는 것이 중요하다.

---

## 6. Background Process와 nohup

프로그램을 백그라운드로 실행하는 방법을 실습했다.

    sleep 600 &

프로세스 확인:

    ps aux | grep '[s]leep 600'

PID를 확인한 후 종료:

    kill <PID>

터미널을 종료해도 프로세스를 계속 실행해야 하는 경우에는
nohup을 사용할 수 있다.

    nohup sleep 600 >/tmp/sleep.log 2>&1 &

nohup은 터미널 종료 등의 상황에서도
프로세스가 계속 실행될 수 있도록 하는 명령이다.

---

## 7. Nginx와 Reverse Proxy

Nginx는 웹 서버뿐만 아니라
Reverse Proxy 역할도 수행할 수 있다.

외부 요청을 Nginx가 받은 후
내부 애플리케이션으로 전달하는 구조다.

외부 Client
→ Nginx
→ 내부 Application

예를 들어 Nginx가 9001 Port에서 요청을 받고
내부의 8002 Port 애플리케이션으로 전달할 수 있다.

    location / {
        proxy_pass http://127.0.0.1:8002;
    }

이러한 구조를 Reverse Proxy라고 한다.

---

## 8. Nginx 설정 확인과 Reload

Nginx 설정을 변경한 경우
먼저 설정 문법을 검사한다.

    nginx -t

문법에 문제가 없다면 설정을 다시 적용한다.

    systemctl reload nginx

Reload는 서비스를 완전히 종료하지 않고
변경된 설정을 다시 적용할 때 사용한다.

운영 환경에서는 설정을 변경한 뒤
바로 Reload하기보다 먼저 nginx -t를 통해
설정 오류 여부를 확인하는 것이 중요하다.

---

## 9. Docker에서 Nginx 확인

Docker에서 Nginx 컨테이너를 실행했다.

컨테이너 확인:

    docker ps

컨테이너 내부 접속:

    docker exec -it web-practice /bin/sh

컨테이너 내부에서 Nginx가 제공하는 웹 페이지를 확인했다.

    wget -qO- http://127.0.0.1:80

Mac 호스트에서는 다음 명령으로 확인했다.

    curl http://localhost:8080

Nginx의 기본 Welcome 페이지가 출력되는 것을 확인했다.

Docker Port 확인:

    docker port web-practice

결과:

    80/tcp -> 0.0.0.0:8080
    80/tcp -> [::]:8080

이는 컨테이너의 80번 Port가
호스트의 8080번 Port와 연결되어 있다는 의미다.

즉,

Mac localhost:8080
→ Docker Host
→ Container:80
→ Nginx

구조로 연결된다.

---

## 10. Docker Nginx 로그 확인

컨테이너의 로그를 확인했다.

    docker logs --tail 10 web-practice

정상적인 HTTP 요청은 다음과 같이 기록된다.

    "GET / HTTP/1.1" 200

HTTP 상태 코드 200은 요청이 정상적으로 처리되었다는 의미다.

또한 favicon.ico가 존재하지 않아
404 오류가 발생한 것도 확인했다.

    open() "/usr/share/nginx/html/favicon.ico" failed

이는 브라우저가 웹 페이지의 favicon을 요청했지만
해당 파일이 컨테이너 내부에 존재하지 않아 발생한 오류다.

전체 서비스 장애라기보다는
특정 파일이 존재하지 않아 발생한 404 오류로 판단할 수 있다.

---

## 11. 로그 분석

서버 로그는 장애 원인을 찾는 데 중요한 자료다.

로그 디렉터리 확인:

    ls -lh /var/log

syslog에서 오류 패턴 검색:

    sudo grep -iE 'error|fail|failed|warning' /var/log/syslog

ERROR 개수 확인:

    sudo grep -i 'error' /var/log/syslog | wc -l

실습 환경에서는 수천 개의 ERROR 로그가 존재하는 것을 확인했다.

따라서 로그를 단순히 많이 읽는 것보다
반복적으로 발생하는 오류 패턴을 집계하는 것이 중요하다.

---

## 12. 오류 패턴 집계

오류 유형을 빈도순으로 확인하기 위해
grep, sed, sort, uniq를 Pipeline으로 연결했다.

    sudo grep -iE 'error|fail|failed|warning' /var/log/syslog \
    | sed 's/.*: //' \
    | sort \
    | uniq -c \
    | sort -nr \
    | head -20

각 명령어의 역할:

grep
→ ERROR, FAIL, FAILED, WARNING 등의 로그 추출

sed
→ 불필요한 앞부분을 제거하고 오류 메시지만 추출

sort
→ 같은 오류를 서로 모아서 정렬

uniq -c
→ 동일한 오류가 몇 번 발생했는지 집계

sort -nr
→ 발생 횟수가 많은 순서로 정렬

head
→ 상위 결과만 출력

실습 결과 반복적으로 발생한 오류 중에는
실습용 반복 오류와 서비스 실패,
Address already in use,
No such file or directory,
division by zero 등의 오류가 확인되었다.

특히 실습 환경에서는 여러 교육생이 사용하는 서버이기 때문에
로그 분석 결과에 다른 교육생의 실습 과정에서 발생한 오류까지
함께 포함될 수 있다는 점도 확인했다.

따라서 실제 운영 환경에서는
로그의 시간, 서비스명, PID, 사용자, 호스트 등을 함께 확인하여
장애의 범위를 구분하는 것이 중요하다.

---

## 13. 장애 분석 사고방식

오늘 네트워크와 서비스 실습을 통해
장애가 발생했을 때 바로 해결책부터 실행하는 것이 아니라
어디에서 문제가 발생했는지 단계적으로 좁혀야 한다는 것을 배웠다.

기본적인 확인 순서는 다음과 같다.

서비스 실행 여부
→ 프로세스 확인
→ Port Listening 확인
→ 서버 방화벽 확인
→ 클라우드 보안 그룹 확인
→ 외부 연결 확인
→ 서비스 로그 확인

핵심은

"어디까지 정상인가?"

를 먼저 확인하고

"어디에서 끊겼는가?"

를 찾는 것이다.

---

## 14. 오늘의 핵심 명령어

    ss -tlnp
    nc -zv <IP> <PORT>

    ps aux
    top
    kill <PID>

    nohup <command> &

    systemctl status <service>
    journalctl -u <service>

    nginx -t
    systemctl reload nginx

    docker ps
    docker exec -it <container> /bin/sh
    docker logs --tail 10 <container>
    docker port <container>

    grep
    sed
    sort
    uniq -c
    wc -l
    head

    ufw status
    ufw allow <PORT>/tcp

---

## 15. 학습 정리

오늘은 Linux 서버에서 네트워크와 서비스를 확인하는
기본적인 운영 흐름을 실습했다.

IP와 Port의 관계,
서비스의 Binding과 Listening,
방화벽과 외부 연결의 관계를 확인하면서
네트워크 장애를 단계적으로 좁혀가는 방법을 익혔다.

또한 systemctl과 journalctl을 이용해
서비스의 상태와 로그를 확인하고,
nohup을 이용해 백그라운드 프로세스를 실행하는 방법도 실습했다.

Nginx에서는 웹 서버와 Reverse Proxy의 개념을 확인했고,
Docker 환경에서는 Nginx 컨테이너를 실행하여
Port Mapping, HTTP 요청, 컨테이너 로그까지 직접 확인했다.

마지막으로 grep, sed, sort, uniq를 조합하여
대량의 로그에서 반복되는 오류 패턴을 집계하면서
단순히 오류가 존재한다는 사실보다
"어떤 오류가 얼마나 자주 발생하는가"를 파악하는 것이
장애 분석에서 중요하다는 것을 배웠다.

오늘 학습의 핵심은

서비스
→ Port
→ 방화벽
→ 네트워크 연결
→ 로그
→ 원인 분석

이라는 전체 흐름을 이해하는 것이었다.
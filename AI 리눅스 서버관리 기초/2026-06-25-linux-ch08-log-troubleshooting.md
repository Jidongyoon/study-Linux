# Linux Chapter 8 - 로그 분석과 트러블슈팅 (2026-06-25)

## 학습 목표
- Linux 로그 분석 기초
- 장애 재현 및 원인 분석
- 웹 서버 및 DB 서버 로그 확인
- 서비스 상태와 포트 점유 확인
- 성능 테스트 및 부하 테스트 기초

---

# 1. locate와 파일 검색

## locate
- 파일 이름 데이터베이스를 이용하여 매우 빠르게 검색
- 새 파일은 `updatedb` 실행 전까지 검색되지 않음

```bash
touch test.txt
locate test.txt
updatedb
locate test.txt
```

### 비교

- locate : 빠르지만 DB 기반
- find : 느리지만 실시간 검색

---

# 2. C 프로그램과 공유 라이브러리

간단한 C 프로그램 작성 후 컴파일

```bash
gcc -o hello hello.c
```

실행 파일이 사용하는 공유 라이브러리 확인

```bash
ldd hello
```

### 학습 내용

- gcc : 컴파일러
- printf() : libc 라이브러리 함수
- ldd : 실행 파일이 사용하는 공유 라이브러리 확인

---

# 3. journalctl 로그 분석

부팅 이력 확인

```bash
journalctl --list-boots
```

현재 커널 로그

```bash
dmesg
```

이전 부팅 로그

```bash
journalctl -b -1
```

커널 로그만 확인

```bash
journalctl -k -b -1
```

---

# 4. 로그 조회

실시간 로그

```bash
journalctl -f
```

최근 5분 로그

```bash
journalctl --since "5 minutes ago"
```

오늘 로그

```bash
journalctl --since today
```

어제 로그

```bash
journalctl --since yesterday
```

에러 로그

```bash
journalctl -p err
```

경고 로그

```bash
journalctl -p warning
```

---

# 5. SSH 로그 분석

서비스 상태

```bash
service sshd status
```

로그 확인

```bash
journalctl -u sshd
```

### 확인한 내용

- 로그인 성공
- 로그인 실패
- 잘못된 비밀번호
- root 로그인 제한
- PermitRootLogin 설정 확인

---

# 6. 로그 용량 관리

로그 사용량 확인

```bash
journalctl --disk-usage
```

로그 디렉터리 확인

```bash
du -sh /var/log
```

30일 이전 로그 삭제

```bash
journalctl --vacuum-time=30d
```

---

# 7. nginx 로그 분석

Access Log

```bash
tail -f /var/log/nginx/access.log
```

HTTP 상태코드 확인

- 200 OK
- 404 Not Found
- 500 Internal Server Error
- 502 Bad Gateway

---

# 8. FastAPI 성능 테스트

동일 요청 반복

```bash
for i in {1..100}
do
    curl http://localhost/analysis/pytorch/pytorch
done
```

Apache Bench

```bash
ab -n 100 -c 100
```

### 비교

- main_perf_time
- main_async

비동기 처리 성능 비교

---

# 9. Apache 권한 문제 재현

서비스 실행

```bash
service httpd start
```

index.html 검색

```bash
find /usr -name index.html
```

권한 확인

```bash
getfacl
```

권한 변경

```bash
chmod o-x
```

### 확인

403 Forbidden 발생 원인 분석

---

# 10. 네트워크 장애 재현

iptables 이용

```bash
iptables -A OUTPUT -p tcp --dport 80 -j DROP
```

확인

```bash
curl www.google.com
```

패키지 다운로드 실패

```bash
dnf check-update
```

### 확인한 내용

- Curl Error (28)
- Timeout
- 네트워크 차단
- 장애 재현 및 원인 분석

---

# 11. MariaDB 장애 분석

서비스 상태

```bash
service mariadb status
```

로그

```bash
journalctl -xeu mariadb.service
```

포트 확인

```bash
netstat -tplan
```

동시 접속 확인

```sql
SHOW VARIABLES LIKE 'max_connections';
```

부하 테스트

```bash
for i in {1..200}; do
    mysql -u testuser -ptestpass testdb &
done
```

---

# 오늘 가장 중요했던 개념

## 장애 분석 순서

```
문제 발생

↓

장애 재현

↓

로그 확인

↓

시스템 상태 확인

↓

원인 분석

↓

복구

↓

정상 동작 확인
```

---

## 자주 사용한 명령어

```bash
journalctl
tail -f
service
systemctl
curl
ab
watch
df
du
find
locate
updatedb
ldd
gcc
iptables
netstat
ps
getfacl
chmod
mysql
```

---

## 느낀 점

오늘은 Linux 로그 분석과 트러블슈팅 흐름을 집중적으로 학습했다.

단순히 명령어를 외우는 것이 아니라,
장애를 직접 재현하고 로그를 통해 원인을 찾는 과정을 반복하며
현업에서 사용하는 문제 해결 방식의 흐름을 이해하는 데 집중했다.

앞으로도 "증상이 아니라 근거(로그)를 보고 판단한다."는 습관을 계속 가져가야겠다.
# Linux Filesystem / Inode / I/O 실습 정리
> 학습일: 2026-06-24

---

# 1. Buffered I/O 와 Direct I/O

## Buffered I/O

일반적인 Linux 파일 입출력 방식

동작 순서

```text
Application
 ↓
Page Cache (메모리)
 ↓
Disk
```

### Write

```text
1. 페이지 캐시 확보
2. 데이터 메모리에 기록
3. Dirty Page 표시
4. Writeback 스레드가 나중에 디스크 반영
```

### Read

```text
1. 페이지 캐시 확인
2. Cache Hit
   → 메모리에서 즉시 읽음

3. Cache Miss
   → 디스크 읽기 요청
   → IRQ 발생
   → 프로세스 Wakeup
   → 데이터 반환
```

### 특징

- 일반적인 Linux 기본 I/O
- Page Cache 활용
- 반복 읽기 성능 우수

---

## Direct I/O

동작 순서

```text
Application
 ↓
Disk
```

### 특징

- Page Cache 사용 안함
- 디스크 직접 접근
- 대용량 DB에서 자주 사용
- 캐시 오염(Cache Pollution) 방지

대표 사례

- Oracle
- MySQL(InnoDB)
- PostgreSQL 일부 환경

---

# 2. Writeback

Buffered I/O에서 사용되는 방식

```text
Application
 ↓
Page Cache 기록
 ↓
Dirty Page 표시
 ↓
주기적으로 Disk 반영
```

장점

- 디스크 I/O 감소
- 성능 향상

주의

- Writeback 전에 장애 발생 시 데이터 유실 가능

---

# 3. Linux 파일 디스크립터(FD)

프로세스는 기본적으로 3개의 FD를 가진다.

| FD | 이름 | 설명 |
|------|------|------|
| 0 | stdin | 표준 입력 |
| 1 | stdout | 표준 출력 |
| 2 | stderr | 표준 에러 |

---

## 예시

### stdout

```bash
echo hello
```

출력

```text
hello
```

---

### stderr

```bash
cat nofile.txt
```

출력

```text
No such file or directory
```

---

# 4. stdout 과 stderr

파이프(|)는 기본적으로 stdout만 전달한다.

예시

```bash
find / -name "*http*" | grep http
```

동작

```text
stdout
 ↓
grep

stderr
 ↓
터미널
```

따라서 Permission denied는 grep이 처리하지 못한다.

---

## stderr 제거

```bash
find / -name "*http*" 2>/dev/null
```

설명

```text
stderr
 ↓
쓰레기통(/dev/null)
```

---

## stderr + stdout 합치기

```bash
find / -name "*http*" 2>&1
```

설명

```text
stderr → stdout
```

---

## Permission 제외 출력

```bash
find / -name "*http*" 2>&1 | grep -v Permission
```

---

# 5. Inode 개념

Linux 파일은 실제 데이터 외에도 메타데이터를 가진다.

Inode 저장 정보

```text
파일 크기
소유자
권한
생성시간
수정시간
데이터 블록 위치
```

저장하지 않는 것

```text
파일 이름
```

---

# 6. Inode Full 실습

1GB Loop Device 생성

```bash
dd if=/dev/zero of=inode_test.img bs=1M count=1000
mkfs.ext4 inode_test.img
mount -o loop inode_test.img /mnt/inode_test
```

확인

```bash
df -i /mnt/inode_test
```

결과

```text
64000 Inodes
```

---

## Inode 고갈 테스트

파일 생성 반복

```c
for (i=0; i<2000000; i++)
{
    fopen(filename, "w");
}
```

결과

```text
failed fd 63987
```

확인

```bash
df -i /mnt/inode_test
```

```text
IFree 0
IUse% 100%
```

디스크 공간은 남아있지만 파일 생성 불가

원인

```text
Inode 고갈
```

---

# 7. du 명령어

디렉터리 용량 확인

```bash
du -sh /usr
```

---

하위 폴더 용량 확인

```bash
du -h /usr/*
```

---

큰 순서 정렬

```bash
du -h /usr/* | sort -rh
```

---

TOP 10 확인

```bash
du -h /usr/* | sort -rh | head -10
```

실무 활용

```text
디스크 사용량 분석
용량 부족 원인 파악
```

---

# 8. gzip 과 tar

## gzip

압축

```bash
gzip file.log
```

결과

```text
file.log.gz
```

원본 제거

---

## gunzip

압축 해제

```bash
gunzip file.log.gz
```

---

## gzip -c

원본 유지

```bash
gzip -c file.log > file.log.gz
```

결과

```text
원본 유지
압축파일 생성
```

---

# 9. tar

파일 묶기

```bash
tar -cvf cache.tar /var/cache
```

특징

```text
압축 X
묶기 O
```

---

압축 포함

```bash
tar -zcvf cache.tar.gz /var/cache
```

---

해제

```bash
tar -zxvf cache.tar.gz
```

---

# 10. 디스크 I/O 모니터링

## sysbench

I/O 부하 생성

```bash
sysbench fileio \
--file-total-size=1G \
--file-test-mode=rndrw \
--time=60 \
--threads=4 run
```

---

## iotop

실시간 I/O 확인

```bash
sudo iotop
```

확인 가능

```text
누가 디스크를 읽는지
누가 디스크를 쓰는지
```

---

## iostat

디스크 전체 상태 확인

```bash
iostat
```

확인 가능

```text
TPS
Read
Write
IO Wait
```

실무 활용

```text
디스크 병목 진단
성능 분석
트러블슈팅
```

---

# 오늘 배운 핵심

```text
Buffered I/O
Direct I/O
Writeback
Page Cache
File Descriptor
stdin/stdout/stderr
2>/dev/null
2>&1
Inode
Inode Full
du
gzip
tar
sysbench
iotop
iostat
```
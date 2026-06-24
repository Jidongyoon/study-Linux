# Linux Chapter 07 - 디스크 · 파일 · 권한 문제 진단

## 학습 목표

리눅스 서버 운영 과정에서 자주 발생하는 디스크, 파일, 권한 문제를 진단하고 해결하는 방법을 학습하였다.

---

## 학습 내용

### 1. 파일 권한(Permission)

리눅스 파일 권한은 다음 3가지 사용자 범위로 구분된다.

| 구분 | 설명 |
|--------|--------|
| Owner | 파일 소유자 |
| Group | 파일 그룹 |
| Others | 기타 사용자 |

권한 종류

| 권한 | 의미 | 숫자 |
|--------|--------|--------|
| r | Read | 4 |
| w | Write | 2 |
| x | Execute | 1 |

예시

```bash
chmod 755 file.sh
chmod u=rwx file.sh
chmod g+w file.sh
chmod o-r file.sh
```

---

### 2. 권한 확인

```bash
ls -l
getfacl filename
stat filename
```

실습을 통해 Owner, Group, Others 권한 구조를 확인하였다.

---

### 3. 사용자 및 그룹 확인

현재 사용자 확인

```bash
whoami
```

현재 계정의 그룹 확인

```bash
id
groups
```

---

### 4. Permission Denied 문제 분석

실습 예제

```bash
cat /var/log/secure
```

결과

```bash
Permission denied
```

원인

- 파일 소유자가 root
- 일반 사용자 계정은 읽기 권한 없음

확인 명령어

```bash
ls -l /var/log/secure
getfacl /var/log/secure
```

---

### 5. sudo 와 리다이렉션(>) 차이

실패

```bash
sudo echo "test" > /etc/locale.conf
```

성공

```bash
sudo bash -c 'echo "test" > /etc/locale.conf'
```

또는

```bash
echo "test" | sudo tee /etc/locale.conf
```

학습 내용

- sudo는 명령어에만 적용된다.
- 리다이렉션(>)은 현재 쉘 권한으로 수행된다.

---

### 6. Inode 이해

확인 명령어

```bash
stat hello.txt
```

예시

```bash
Inode: 109131467
```

학습 내용

- 파일명과 실제 데이터를 연결하는 메타데이터 구조
- 권한, 소유자, 시간 정보 저장
- 데이터 블록 위치 정보 관리

---

### 7. 파일시스템 구조

파일시스템 구성

```text
Super Block
↓
Inode Block
↓
Data Block
```

### Super Block

파일시스템 전체 정보 저장

- 파일시스템 종류
- 블록 크기
- inode 개수
- 사용량 정보

### Inode Block

파일 메타데이터 저장

- 권한
- 소유자
- 그룹
- 파일 크기
- 시간 정보

### Data Block

실제 파일 데이터 저장

---

### 8. 파일시스템 종류 확인

```bash
df -Th
```

실습 환경

```text
xfs
tmpfs
devtmpfs
vfat
```

---

### 9. 가상 파일시스템

#### /proc

커널 정보를 제공하는 가상 파일시스템

예시

```bash
cat /proc/version
cat /proc/meminfo
cat /proc/cpuinfo
```

특징

- 실제 디스크 파일이 아님
- 커널 상태를 실시간 제공

---

#### /sys

장치 및 커널 정보를 제공하는 가상 파일시스템

---

### 10. Input/output error 분석

실습

```bash
echo "test" > /proc/version
```

결과

```bash
Input/output error
```

원인

- /proc/version 은 일반 파일이 아님
- procfs의 특수 파일
- 커널 정책상 쓰기 불가

---

### 11. 경로(Path) 권한 문제

파일 권한이 있어도 상위 디렉토리 접근 권한이 없으면 접근 불가

예시

```bash
/home/testuser/Fast-Api-example
```

확인 명령어

```bash
namei -l 경로
```

학습 내용

- 파일 권한
- 디렉토리 접근 권한(x)

을 함께 확인해야 함

---

## 실습 명령어 정리

```bash
ls -l
chmod
chown
getfacl
stat
whoami
id
groups
df -h
df -Th
find
namei
```

---

## 학습 회고

권한 문제는 단순히 chmod로 해결하는 것이 아니라 파일 소유자, 그룹, 상위 디렉토리 권한, 파일시스템 종류, 실행 주체 계정을 함께 고려해야 한다는 점을 이해하였다.

또한 inode, super block, data block 구조와 procfs, tmpfs 같은 가상 파일시스템의 개념을 학습하였다.

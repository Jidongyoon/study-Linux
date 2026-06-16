# 2026-06-16 학습 정리

## 1. cloc(Code Line Of Code)

오픈소스 프로젝트의 언어별 코드량을 분석하는 도구

```bash
cloc --json <repository>
```

### 활용

* 프로젝트 규모 파악
* 주 사용 언어 확인
* 프로젝트 성격 추론

예시

| 언어         | 추론               |
| ---------- | ---------------- |
| Go         | 백엔드, 클라우드, 인프라   |
| TypeScript | 웹 서비스, 프론트엔드/백엔드 |
| Python     | AI, 자동화, 웹 API   |
| C          | 운영체제, 시스템        |
| C++        | 게임엔진, AI 프레임워크   |

---

## 2. JSON 파싱(Parsing)

파싱이란 문자열 데이터를 프로그램이 사용할 수 있는 형태로 변환하는 과정이다.

예시

```python
cloc_json = json.loads(cloc_result.stdout)
```

* cloc 출력(JSON 문자열)
* Python Dictionary 객체로 변환

---

## 3. FastAPI + GitHub Repository 분석

처리 흐름

```text
요청
 ↓
git clone
 ↓
cloc 분석
 ↓
JSON 파싱
 ↓
응답 반환
```

---

## 4. 성능 측정

명령어 실행 시간 측정

```python
start = time.time()

run_command(...)

elapsed = time.time() - start
```

실험 결과

### FastAPI Repository

```text
git clone : 2.2초
cloc : 1.2초
```

### PyTorch Repository

```text
git clone : 11.1초
cloc : 13.9초
```

---

## 5. Nginx Timeout

구조

```text
Browser
 ↓
Nginx
 ↓
FastAPI
```

문제점

* FastAPI 작업이 오래 걸리면
* Nginx가 먼저 Timeout 발생 가능

예시

```text
Nginx Timeout : 10초
FastAPI 작업 : 25초
```

결과

```text
사용자 → 504 Gateway Timeout

하지만 FastAPI는 계속 작업 수행
```

---

## 6. Redis(Valkey) 캐시

처리 흐름

```text
요청
 ↓
Redis 조회

Cache Hit
 ↓
즉시 응답

Cache Miss
 ↓
git clone
 ↓
cloc
 ↓
Redis 저장
 ↓
응답
```

장점

* 동일 요청 재분석 방지
* 응답 속도 향상
* 서버 부하 감소

실습 결과

```text
첫 요청
git clone + cloc 수행

두 번째 요청
Redis Cache Hit
즉시 응답
```

---

## 7. Python 들여쓰기

Python에서는 들여쓰기가 문법이다.

잘못된 예

```python
if condition:
print("hello")
```

올바른 예

```python
if condition:
    print("hello")
```

대표 에러

```text
IndentationError
```

---

## 오늘 핵심

* cloc로 오픈소스 구조 분석
* JSON 파싱 이해
* FastAPI 요청 처리 흐름 이해
* 성능 측정 방법 학습
* Nginx Timeout 원인 분석
* Redis(Valkey) 캐시 적용
* Python 들여쓰기 규칙 이해

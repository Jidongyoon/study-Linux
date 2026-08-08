# 2026-08-08 FastAPI + MySQL 연동

## 오늘 배운 것

### MySQL
- `SELECT` : 조회
- `INSERT` : 추가
- `UPDATE` : 수정
- `DELETE` : 삭제
- `WHERE` : 조건
- `AND / OR` : 여러 조건
- `ORDER BY` : 정렬
- `AUTO_INCREMENT` : ID 자동 증가
- `PRIMARY KEY` : 고유한 식별자

### SQLAlchemy / PyMySQL
- **SQLAlchemy** : Python에서 DB를 다루기 쉽게 해주는 라이브러리
- **PyMySQL** : Python과 MySQL을 연결하는 드라이버
- **Engine** : DB 연결 관리
- **Session** : DB 작업을 위한 작업 공간
- **SessionLocal** : Session을 만들어주는 객체
- **Base** : ORM Model의 기준

전체 구조:

Python / FastAPI
→ SQLAlchemy
→ PyMySQL
→ MySQL

---

## MySQL 설정

DB:

`lg_twins_db`

테이블:

`players`

선수 데이터:

- 홍창기 / OF
- 오스틴 딘 / 1B
- 문보경 / 3B

FastAPI 전용 사용자도 생성했다.

`fastapi@localhost`

그리고 `lg_twins_db`에 접근할 권한을 부여했다.

---

## database.py

`app/database.py`에서 SQLAlchemy와 MySQL 연결을 설정했다.

핵심 구성:

- `DATABASE_URL` → DB 접속 정보
- `create_engine()` → Engine 생성
- `sessionmaker()` → Session 생성기
- `declarative_base()` → ORM Model의 기준

DB 연결 주소:

`mysql+pymysql://fastapi:비밀번호@localhost/lg_twins_db`

---

## 실제 DB 조회 테스트

`test_db.py`에서 Session을 생성하고 SQL을 직접 실행했다.

`SELECT * FROM players`

실행 결과:

- 홍창기
- 오스틴 딘
- 문보경

Python에서 MySQL의 실제 데이터를 가져오는 데 성공했다.

최종 흐름:

Python
→ SQLAlchemy Session
→ PyMySQL
→ MySQL
→ players
→ Python

---

## 트러블슈팅

### 1. SQLAlchemy를 찾지 못하는 오류

에러:

`ModuleNotFoundError: No module named 'sqlalchemy'`

원인:

SQLAlchemy는 `venv`에 설치되어 있었지만 `venv`를 활성화하지 않고 시스템 Python으로 실행했다.

해결:

`source venv/bin/activate`

확인:

`which python`

결과가 프로젝트의 `venv/bin/python`을 가리키는지 확인했다.

**배운 점:** Python 패키지 오류가 발생하면 코드뿐만 아니라 현재 사용 중인 Python 환경도 확인해야 한다.

---

### 2. MySQL 로그인 거부

에러:

`Access denied for user 'fastapi'@'localhost'`

원인:

MySQL의 `fastapi` 계정 비밀번호와 Python의 `DATABASE_URL`에 입력한 비밀번호가 일치하지 않았다.

해결:

MySQL에서 `fastapi` 사용자의 비밀번호를 다시 설정하고 재실행했다.

결과적으로 Python에서 MySQL 데이터를 정상적으로 조회했다.

**배운 점:** DB 연결 오류가 발생하면 사용자, 비밀번호, host, database, 권한 등을 확인해야 한다.

---

## 오늘의 핵심

오늘은 단순히 MySQL을 사용하는 것에서 끝나지 않고,

**Python에서 MySQL 데이터를 실제로 가져오는 과정**까지 성공했다.

특히 기억할 것:

- SQLAlchemy = DB를 Python에서 다루기 쉽게 해주는 도구
- PyMySQL = Python과 MySQL의 통신을 담당
- Engine = DB 연결 관리
- Session = DB 작업 단위
- venv = 프로젝트별 Python 실행 환경

---

## 다음 학습

다음에는 SQL을 직접 작성하는 방식에서 한 단계 더 나아가

**SQLAlchemy ORM Model을 만들어 `players` 테이블과 연결한다.**

목표:

FastAPI
→ SQLAlchemy ORM
→ MySQL
→ players
→ JSON 응답

최종적으로 FastAPI의 `/players` API에서 MySQL의 선수 목록을 가져오도록 구현한다.
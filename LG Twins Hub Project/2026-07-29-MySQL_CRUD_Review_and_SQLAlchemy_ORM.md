# Day 10 - MySQL CRUD 복습 및 SQLAlchemy ORM 입문

## 📅 학습 목표

- MySQL CRUD(Query) 마무리
- UPDATE / DELETE 심화
- AUTO_INCREMENT 동작 원리 이해
- ORM(Object Relational Mapping) 개념 이해
- SQLAlchemy 및 PyMySQL 설치

---

# 1. MySQL CRUD 복습

## UPDATE

데이터를 수정할 때 사용하는 명령어이다.

```sql
UPDATE players
SET position = 'DH'
WHERE id = 3;
```

### 실행 순서

1. players 테이블 찾기
2. WHERE 조건 검색
3. 조건에 맞는 행 선택
4. position 값을 DH로 변경

---

## 여러 컬럼 수정

```sql
UPDATE players
SET
    name='오스틴 딘',
    position='1B'
WHERE id=3;
```

한 번의 UPDATE로 여러 컬럼을 수정할 수 있다.

---

## WHERE 없이 UPDATE

```sql
UPDATE players
SET position='DH';
```

### 결과

모든 선수의 position이 DH로 변경된다.

따라서 UPDATE를 사용할 때는 WHERE 조건을 반드시 확인해야 한다.

---

# DELETE

특정 데이터 삭제

```sql
DELETE FROM players
WHERE id=4;
```

조건에 맞는 데이터만 삭제된다.

---

모든 데이터 삭제

```sql
DELETE FROM players;
```

WHERE가 없으면 테이블의 모든 데이터가 삭제된다.

---

# AUTO_INCREMENT

AUTO_INCREMENT는 삭제된 번호를 다시 사용하지 않는다.

예시

```
1
2
3
4
```

id=4 삭제

↓

새로운 데이터 추가

```
1
2
3
5
```

다음 번호가 계속 증가한다.

---

# SELECT 복습

정렬

```sql
SELECT *
FROM players
ORDER BY id DESC;
```

내림차순으로 정렬한다.

---

LIMIT

```sql
SELECT *
FROM players
ORDER BY id DESC
LIMIT 2;
```

정렬 후 상위 2개의 데이터만 조회한다.

---

# 오늘 이해한 핵심

- WHERE → 먼저 조건 검색
- ORDER BY → 조건 결과 정렬
- LIMIT → 필요한 개수만 반환

실행 순서를 이해하는 것이 중요하다.

---

# ORM(Object Relational Mapping)

ORM은 Python과 MySQL 사이를 연결하는 기술이다.

Python 코드를 SQL로 변환하여 MySQL이 이해할 수 있도록 도와준다.

구조

```
Python
    │
    ▼
SQLAlchemy (ORM)
    │
    ▼
MySQL
```

Python은 Python 문법만 이해하고,

MySQL은 SQL 문법만 이해하기 때문에

ORM이 중간에서 번역기 역할을 수행한다.

---

# SQLAlchemy

Python에서 가장 많이 사용하는 ORM 라이브러리이다.

예시

Python 코드

```python
player = Player(name="문보경", position="3B")
```

↓

내부적으로

```sql
INSERT INTO players (name, position)
VALUES ('문보경', '3B');
```

로 변환된다.

---

# PyMySQL

PyMySQL은 Python과 MySQL 서버를 실제로 연결하는 드라이버이다.

비유

```
FastAPI
    │
    ▼
SQLAlchemy (운전자)
    │
    ▼
PyMySQL (자동차)
    │
    ▼
MySQL
```

SQLAlchemy가 SQL을 생성하고

PyMySQL이 MySQL 서버까지 전달한다.

---

# 설치

```bash
pip install sqlalchemy pymysql
```

설치 확인

```bash
pip show sqlalchemy
pip show pymysql
```

## 설치 결과

- SQLAlchemy 2.0.51
- PyMySQL 1.2.0
- greenlet 3.2.5

프로젝트의 venv에 정상적으로 설치 완료.

---

# 오늘 배운 핵심 정리

- UPDATE를 이용한 데이터 수정
- DELETE를 이용한 데이터 삭제
- WHERE의 중요성
- AUTO_INCREMENT 동작 원리
- ORDER BY와 LIMIT 실행 순서
- ORM의 역할 이해
- SQLAlchemy와 PyMySQL의 역할 이해
- SQLAlchemy 및 PyMySQL 설치 완료

---

# 다음 학습

- database.py 생성
- create_engine()
- Database URL
- SessionLocal
- Base
- FastAPI와 MySQL 연결
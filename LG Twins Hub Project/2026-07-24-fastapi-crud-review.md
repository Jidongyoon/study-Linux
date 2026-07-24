# FastAPI CRUD 복습

> 날짜 : 2026-07-24

## 📌 오늘 목표

10일 정도 공부를 쉬어서 FastAPI 프로젝트를 다시 실행하고,
CRUD(Create, Read, Update, Delete) 개념을 복습하였다.

---

# 1. 프로젝트 실행

가상환경 활성화

```bash
source venv/bin/activate
```

FastAPI 서버 실행

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Swagger 접속

```
http://192.168.111.100:8000/docs
```

---

# 2. GET (조회)

## 개념

데이터를 조회하는 기능이다.

기존 데이터를 변경하지 않는다.

```python
@app.get("/players")
```

요청

```
GET /players
```

동작

```
브라우저

↓

FastAPI

↓

player_list 반환

↓

브라우저 출력
```

---

# 3. POST (생성)

## 개념

새로운 데이터를 추가하는 기능이다.

```python
@app.post("/players")
```

요청 예시

```json
{
    "name":"문보경",
    "position":"3B"
}
```

동작 순서

```
브라우저

↓

Player(BaseModel) 검증

↓

player 객체 생성

↓

player.model_dump()

↓

append()

↓

player_list에 추가
```

---

## Player와 player의 차이

### Player

설계도(Class)

```python
class Player(BaseModel):
```

### player

실제 생성된 객체

```python
player: Player
```

---

## model_dump()

객체(Object)를 Dictionary로 변환한다.

```
player

↓

player.model_dump()

↓

{
    "name":"문보경",
    "position":"3B"
}
```

---

## append()

리스트의 마지막에 데이터를 추가한다.

```python
player_list.append(player.model_dump())
```

---

# 4. PUT (수정)

## 개념

기존 데이터를 수정한다.

새로운 데이터를 만드는 것이 아니라 기존 데이터를 변경한다.

```python
@app.put("/players/{player_id}")
```

요청

```
PUT /players/1
```

JSON

```json
{
    "name":"문보경",
    "position":"3B"
}
```

동작

```
player_id 확인

↓

Player 검증

↓

player.model_dump()

↓

player_list[player_id] 교체
```

핵심 코드

```python
player_list[player_id] = player.model_dump()
```

---

# 5. DELETE (삭제)

## 개념

기존 데이터를 삭제한다.

```python
@app.delete("/players/{player_id}")
```

요청

```
DELETE /players/1
```

핵심 코드

```python
del player_list[player_id]
```

삭제 후에는 뒤에 있는 데이터가 앞으로 당겨진다.

예시

삭제 전

```
0 홍창기
1 문보경
2 박동원
```

삭제 후

```
0 홍창기
1 박동원
```

---

# 6. 예외 처리

존재하지 않는 번호를 수정하거나 삭제하지 못하도록 예외 처리하였다.

```python
if player_id < 0 or player_id >= len(player_list):
    raise HTTPException(
        status_code=404,
        detail="선수를 찾을 수 없습니다."
    )
```

### len()

리스트의 길이를 반환한다.

### HTTPException

FastAPI에서 HTTP 오류를 발생시키는 클래스이다.

404는 "찾을 수 없음(Not Found)"을 의미한다.

---

# 7. CRUD 정리

| HTTP Method | 기능 | Python 코드 |
|--------------|------|-------------|
| GET | 조회(Read) | `return player_list` |
| POST | 생성(Create) | `player_list.append(player.model_dump())` |
| PUT | 수정(Update) | `player_list[player_id] = player.model_dump()` |
| DELETE | 삭제(Delete) | `del player_list[player_id]` |

---

# 오늘 배운 핵심

- REST API의 CRUD(Create, Read, Update, Delete)를 구현하였다.
- Player(설계도)와 player(객체)의 차이를 이해하였다.
- model_dump()를 사용하여 객체를 Dictionary로 변환하였다.
- append()를 사용하여 리스트에 데이터를 추가하였다.
- PUT을 사용하여 기존 데이터를 수정하였다.
- DELETE를 사용하여 데이터를 삭제하였다.
- HTTPException과 404 예외 처리를 구현하였다.

---

# 다음 목표

- MySQL 설치
- FastAPI + MySQL 연동
- RAM(player_list) 대신 Database에 데이터 저장하기


# FastAPI POST API 구현 및 데이터 추가 실습

## 📅 학습일

2026-07-11

---

# 오늘의 목표

GET으로 조회만 하던 API에 POST 기능을 추가하여 새로운 선수를 등록하는 기능을 구현했다.

---

# 학습 내용

## 1. BaseModel(Pydantic)

```python
class Player(BaseModel):
    name: str
    position: str
```

### 이해한 내용

- Player는 선수 데이터의 설계도이다.
- name과 position 두 개의 문자열 데이터를 가진다.
- FastAPI는 JSON 데이터를 Player 객체로 변환하여 함수의 매개변수에 전달한다.

---

## 2. player_list 위치 변경

기존에는 player_list가 GET 함수 내부에 있었다.

```python
def players():
    player_list = [...]
```

이 경우 GET 요청이 들어올 때마다 리스트가 새로 생성되므로
POST에서 추가한 데이터가 유지되지 않았다.

수정 후

```python
player_list = [
    ...
]
```

함수 밖으로 이동하여

- GET
- POST

모두 같은 player_list를 사용하도록 변경하였다.

---

## 3. POST API 생성

```python
@app.post("/players")
def create_player(player: Player):
```

### 이해한 내용

POST 요청이 들어오면

- FastAPI가 JSON 데이터를 읽는다.
- Player 설계도와 비교한다.
- Player 객체를 생성한다.
- create_player() 함수의 player 매개변수로 전달한다.

---

## 4. model_dump()

```python
player.model_dump()
```

### 이해한 내용

Player 객체를 Dictionary 형태로 변환한다.

변환 전

Player 객체

↓

변환 후

```python
{
    "name": "문보경",
    "position": "3B"
}
```

---

## 5. append()

```python
player_list.append(player.model_dump())
```

### 이해한 내용

변환된 Dictionary를 player_list의 맨 뒤에 추가한다.

추가 전

```python
[
    {"name":"홍창기","position":"OF"},
    {"name":"오스틴","position":"1B"},
    {"name":"박동원","position":"C"}
]
```

추가 후

```python
[
    {"name":"홍창기","position":"OF"},
    {"name":"오스틴","position":"1B"},
    {"name":"박동원","position":"C"},
    {"name":"문보경","position":"3B"}
]
```

---

# POST 요청 흐름

브라우저

↓

JSON 데이터 작성

```json
{
    "name":"문보경",
    "position":"3B"
}
```

↓

FastAPI

↓

Player(BaseModel)

↓

player.model_dump()

↓

player_list.append()

↓

"선수가 추가되었습니다."

↓

브라우저 응답

---

# 오늘 가장 중요하게 이해한 내용

GET는 조회 요청이다.

POST는 새로운 데이터를 서버에 등록하는 요청이다.

FastAPI는 POST 요청으로 받은 JSON 데이터를 Player 객체로 변환하여 함수의 매개변수에 전달한다.

Player 객체는 model_dump()를 통해 Dictionary로 변환한 후 append()를 사용하여 player_list에 추가한다.

---

# 느낀 점

이번 실습을 통해 POST 요청의 흐름을 처음부터 끝까지 이해할 수 있었다.

JSON → Player(BaseModel) → model_dump() → append() → player_list

이 흐름을 이해하면서 GET과 POST의 차이를 명확하게 알게 되었다.
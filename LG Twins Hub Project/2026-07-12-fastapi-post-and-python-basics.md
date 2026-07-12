# FastAPI POST API와 Python 기초 문법 이해

## 📅 학습일

2026-07-12

---

# 오늘의 목표

POST 요청의 동작 원리를 이해하고, FastAPI가 JSON 데이터를 어떻게 처리하는지와 Python 객체 문법을 학습하였다.

---

# 1. GET과 POST의 차이

## GET

- 조회(검색) 요청
- URL(Query Parameter)로 데이터를 전달한다.
- 데이터를 가져오는 역할을 한다.

예시

```
GET /players?position=OF
```

한국말

> 외야수 선수 목록을 보여주세요.

---

## POST

- 등록 요청
- JSON 데이터를 서버에 전달한다.
- 새로운 데이터를 생성한다.

예시

```json
{
    "name":"김현수",
    "position":"OF"
}
```

한국말

> 김현수 선수를 등록해주세요.

---

# 2. Player(BaseModel)

```python
class Player(BaseModel):
    name: str
    position: str
```

## 이해한 내용

Player는 선수 정보를 표현하는 설계도이다.

설계도에는

- 이름(name)
- 포지션(position)

두 개의 문자열이 반드시 존재해야 한다.

---

# 3. 매개변수(Parameter)

```python
def create_player(player: Player):
```

## 이해한 내용

player는 함수가 전달받는 매개변수이다.

FastAPI는

브라우저가 보낸 JSON 데이터를

Player 객체로 만든 후

player 매개변수에 넣어준다.

즉

JSON

↓

Player 객체

↓

player 매개변수

↓

함수 실행

---

# 4. model_dump()

```python
player.model_dump()
```

## 이해한 내용

Player 객체를 Dictionary 형태로 변환한다.

변환 전

Player 객체

↓

변환 후

```python
{
    "name":"김현수",
    "position":"OF"
}
```

---

# 5. append()

```python
player_list.append(player.model_dump())
```

## 이해한 내용

변환된 Dictionary를

player_list의 맨 뒤에 추가한다.

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
    {"name":"김현수","position":"OF"}
]
```

---

# 6. 메모리(RAM)

현재 player_list는 메모리에 저장된다.

따라서

- 서버 실행 중에는 데이터가 유지된다.
- 서버를 재시작하면 데이터는 사라진다.

향후 MySQL을 사용하면 디스크에 저장되어 재시작 후에도 유지된다.

---

# 7. print()와 return()

## print()

```python
print(player.name)
```

개발자가 서버 터미널에서 확인하기 위한 출력이다.

출력 위치

서버 터미널

예시

```
김현수
```

---

## return

```python
return {
    "message":"선수가 추가되었습니다."
}
```

브라우저(클라이언트)에게 응답을 보낸다.

Swagger 응답

```json
{
    "message":"선수가 추가되었습니다."
}
```

---

# 8. Python 기초 문법 복습

## ()

함수 실행

```python
print(player.name)
```

---

## {}

Dictionary

```python
{
    "name":"홍창기",
    "position":"OF"
}
```

---

## []

List

```python
player_list = []
```

---

## =

값 저장

```python
name = "홍창기"
```

---

## :

타입 또는 Key와 Value를 연결

```python
name: str
```

```python
"name":"홍창기"
```

---

## .

객체 안으로 들어간다.

```python
player.name
```

↓

player 안의 name을 가져온다.

```python
player.position
```

↓

player 안의 position을 가져온다.

```python
player.model_dump()
```

↓

player 안의 model_dump() 기능을 실행한다.

---

## ,

여러 값을 구분한다.

---

## ""

문자열(String)

```python
"홍창기"
```

---

# 오늘 가장 중요하게 이해한 내용

Player는 설계도이다.

player는 실제 데이터를 담는 매개변수이다.

FastAPI는 JSON 데이터를 Player 객체로 만든 후

player 매개변수에 전달한다.

player.model_dump()는 Player 객체를 Dictionary로 변환하며

append()를 사용하여 player_list에 추가한다.

print()는 서버 터미널에 출력하며

return은 브라우저(클라이언트)에게 응답을 보낸다.

---

# 느낀 점

오늘은 단순히 POST API를 구현하는 것이 아니라

FastAPI 내부에서 JSON → Player 객체 → Dictionary → List로 이어지는 전체 흐름을 이해하였다.

또한 Python의 핵심 문법인

- ()
- []
- {}
- .
- =
- :

의 의미를 다시 정리하면서 코드를 문장처럼 읽는 연습을 하였다.
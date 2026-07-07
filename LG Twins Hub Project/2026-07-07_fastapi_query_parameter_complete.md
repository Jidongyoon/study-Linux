# FastAPI 학습 기록 (2026-07-07)

## 학습 목표

- Query Parameter 심화
- Python 실행 흐름 이해
- players() 함수 분석
- name 검색 기능 구현
- Query Parameter 마무리

---

# 1. Query Parameter 복습

## 전체 조회

```
/players
```

결과

```json
[
    {
        "name":"홍창기",
        "position":"OF"
    },
    {
        "name":"오스틴",
        "position":"1B"
    },
    {
        "name":"박동원",
        "position":"C"
    }
]
```

---

## Position 검색

```
/players?position=OF
```

결과

```json
[
    {
        "name":"홍창기",
        "position":"OF"
    }
]
```

---

## Name 검색

```
/players?name=홍창기
```

결과

```json
[
    {
        "name":"홍창기",
        "position":"OF"
    }
]
```

---

## 검색 결과 없음

```
/players?position=SS
```

결과

```json
{
    "message":"검색 결과가 없습니다."
}
```

---

# 2. Python 타입 힌트(Type Hint)

```python
def players(position: str = None, name: str = None):
```

## position: str

- 문자열(String)을 전달받는다는 의미

## = None

- 값이 전달되지 않으면 기본값으로 None 사용

---

# 3. players() 함수 실행 흐름

브라우저

↓

```
/players?position=OF
```

↓

FastAPI

↓

```python
players(position="OF", name=None)
```

↓

player_list 생성

↓

for문 반복

↓

if 조건 비교

↓

result.append()

↓

return result

↓

FastAPI가 JSON으로 변환

↓

브라우저 출력

---

# 4. List와 Dictionary 이해

## List

```python
result = []
```

검색 결과를 저장하는 리스트

---

## Dictionary

```python
player = {
    "name":"홍창기",
    "position":"OF"
}
```

Key와 Value 형태로 데이터를 저장

예시

```python
player["name"]
```

↓

```
홍창기
```

```python
player["position"]
```

↓

```
OF
```

---

# 5. for문의 동작

```python
for player in player_list:
```

컴퓨터는

1.

```python
player = {
    "name":"홍창기",
    "position":"OF"
}
```

↓

2.

```python
player = {
    "name":"오스틴",
    "position":"1B"
}
```

↓

3.

```python
player = {
    "name":"박동원",
    "position":"C"
}
```

순서대로 하나씩 꺼내어 검사한다.

---

# 6. append()

```python
result.append(player)
```

검색된 선수를 결과 리스트에 추가한다.

추가 전

```text
[]
```

추가 후

```text
[
    {
        "name":"홍창기",
        "position":"OF"
    }
]
```

---

# 7. return

```python
return result
```

return은 FastAPI에게 결과를 전달한다.

FastAPI는 Python 객체를 JSON으로 자동 변환하여 브라우저에 응답한다.

---

# 8. AND 조건

```python
if position is None and name is None:
```

의미

- position도 없고
- name도 없으면

전체 선수 목록 반환

---

# 9. 오늘 배운 핵심 개념

- Type Hint(str)
- Query Parameter
- List
- Dictionary
- for
- if
- append()
- return
- AND 조건(and)
- FastAPI 실행 흐름

---

# 10. 오늘 느낀 점

오늘은 새로운 기능을 많이 만드는 것보다 Python 코드가 실제로 어떤 순서로 실행되는지 이해하는 데 집중하였다.

특히

- player["name"]
- player["position"]
- result.append()
- return result

가 어떤 의미인지 이해하게 되었고,

브라우저 → FastAPI → Python → JSON → 브라우저

까지의 전체 흐름을 머릿속으로 그려볼 수 있게 되었다.

앞으로 POST와 MySQL을 배우기 전에 Python 실행 흐름을 이해하는 것이 얼마나 중요한지 느낄 수 있었다.

---

# 다음 목표

- Query Parameter 리팩토링
- POST API
- Request Body
- Pydantic(BaseModel)
- 선수 추가 기능 구현
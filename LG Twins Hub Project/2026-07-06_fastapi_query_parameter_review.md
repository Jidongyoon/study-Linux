# FastAPI 학습 기록 (2026-07-06)

## 학습 목표
- Query Parameter 복습
- players() 함수 실행 흐름 이해
- Python 문법(for, if, return) 복습

---

# 1. Query Parameter

기존 API

```python
@app.get("/players")
def players(position: str = None):
```

Query Parameter를 이용하여

```
/players?position=OF
```

형태의 요청을 처리하였다.

브라우저에서 요청하면 FastAPI가 자동으로

```python
position = "OF"
```

으로 값을 전달해 준다는 것을 이해하였다.

---

# 2. players() 함수 구조

```python
@app.get("/players")
def players(position: str = None, name: str = None):
```

- position : 포지션 검색
- name : 이름 검색(구현 예정)

---

# 3. player_list

```python
player_list = [
    {
        "name": "홍창기",
        "position": "OF"
    },
    ...
]
```

선수 정보를 리스트(List) 안에 딕셔너리(Dictionary) 형태로 저장하였다.

---

# 4. if 문

```python
if position is None:
    return player_list
```

position 값이 없으면

전체 선수 목록을 반환하도록 구현하였다.

---

# 5. result 리스트

```python
result = []
```

검색 결과를 저장하기 위한 빈 리스트 생성.

---

# 6. for 문

```python
for player in player_list:
```

player_list 안의 선수들을

- 홍창기
- 오스틴
- 박동원

순서대로 하나씩 꺼내어 검사한다.

---

# 7. 조건 검색

```python
if player["position"] == position:
    result.append(player)
```

검색한 position과 선수의 position이 같으면

검색 결과(result)에 추가한다.

---

# 8. append()

```python
result.append(player)
```

리스트에 검색된 선수를 추가하는 함수.

예시

검색 전

```text
[]
```

검색 후

```text
[
    홍창기
]
```

---

# 9. len()

```python
if len(result) == 0:
```

검색 결과의 개수를 확인한다.

검색 결과가 없으면

```python
return {
    "message": "검색 결과가 없습니다."
}
```

를 반환한다.

---

# 10. 테스트 결과

## 전체 선수 조회

```
/players
```

정상 동작

---

## 포지션 검색

```
/players?position=OF
```

정상 동작

반환 결과

```json
[
    {
        "name":"홍창기",
        "position":"OF"
    }
]
```

---

## 존재하지 않는 포지션

```
/players?position=SS
```

정상 동작

```json
{
    "message":"검색 결과가 없습니다."
}
```

---

# 오늘 느낀 점

FastAPI보다 Python의 실행 흐름(for, if, return)이 아직 익숙하지 않다는 것을 느꼈다.

앞으로는 Python 문법을 조금 더 이해하면서 FastAPI를 함께 학습하면 전체 구조를 이해하는 데 도움이 될 것 같다.

특히 아래 코드의 실행 순서를 이해하는 것이 중요하다고 느꼈다.

```python
result = []

for player in player_list:
    ...

result.append(player)

return result
```

오늘은 API를 구현하는 것보다 코드가 어떤 순서로 실행되는지 이해하는 데 집중하였다.

---

# 다음 목표

- Python List
- Dictionary
- for 문
- if 문
- append()
- Query Parameter(name 검색)
- POST API 시작
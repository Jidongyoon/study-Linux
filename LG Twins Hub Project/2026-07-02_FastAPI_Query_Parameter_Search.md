# FastAPI Query Parameter 검색 기능 구현 및 Python 기초

**날짜:** 2026-07-02

---

## 오늘 목표

- Query Parameter를 활용한 검색 기능 구현
- 검색 결과가 없을 때 메시지 반환
- FastAPI 코드에서 사용되는 Python 문법 이해

---

# 1. Query Parameter 복습

기존 URL

```
/players
```

Query Parameter 사용

```
/players?position=OF
```

FastAPI에서는

```python
def players(position: str = None):
```

처럼 매개변수를 선언하면 URL의 Query Parameter를 자동으로 받아온다.

예시

```
/players?position=OF
```

↓

```python
position = "OF"
```

```
/players
```

↓

```python
position = None
```

---

# 2. 선수 검색 기능 구현

선수 리스트를 반복문으로 순회하면서

검색한 포지션과 같은 선수만 반환하도록 구현하였다.

사용한 문법

```python
for player in player_list:
```

```python
if player["position"] == position:
```

```python
result.append(player)
```

동작 과정

```
브라우저

↓

/players?position=OF

↓

player_list 반복

↓

포지션 비교

↓

조건이 맞는 선수만 result에 추가

↓

결과 반환
```

---

# 3. 검색 결과가 없을 경우 처리

검색 결과가 없는 경우

기존

```json
[]
```

대신

```json
{
    "message": "검색 결과가 없습니다."
}
```

를 반환하도록 구현하였다.

사용 코드

```python
if len(result) == 0:
    return {
        "message": "검색 결과가 없습니다."
    }
```

---

# 4. Python 문법 정리

## def

함수를 만드는 키워드

```python
def players():
```

---

## if

조건문

```python
if position is None:
```

조건이 참일 경우 실행된다.

---

## for

리스트 안의 데이터를 하나씩 꺼내 반복한다.

```python
for player in player_list:
```

---

## append()

리스트에 데이터를 추가한다.

```python
result.append(player)
```

---

## len()

리스트 안의 데이터 개수를 반환한다.

```python
len(result)
```

예시

```
[]

↓

len(result)

↓

0
```

---

## return

함수의 실행을 종료하고 결과를 반환한다.

예시

```python
return player_list
```

```python
return result
```

```python
return {
    "message": "검색 결과가 없습니다."
}
```

---

# 5. Python과 JSON 관계

FastAPI는 Python 객체를 JSON으로 자동 변환한다.

Python

```python
{
    "message": "안녕하세요",
    "success": True
}
```

↓

JSON

```json
{
    "message": "안녕하세요",
    "success": true
}
```

변환되는 값

| Python | JSON |
|---------|------|
| True | true |
| False | false |
| None | null |

---

# 오늘 느낀 점

FastAPI를 공부하면서 Python 문법도 함께 배우고 있다.

처음에는 `def`, `if`, `for`가 어렵게 느껴졌지만,

FastAPI 코드 안에서 실제로 사용해 보니

조금씩 동작 원리를 이해하기 시작했다.

또한 Query Parameter를 이용한 검색 기능을 직접 구현하면서

API가 데이터를 어떻게 검색하고 반환하는지 이해할 수 있었다.

---

# 다음 학습 예정

- Query Parameter 심화
- 이름(name) 검색 기능 추가
- 여러 Query Parameter 동시 사용
- GET 요청 마무리
- POST 요청 시작
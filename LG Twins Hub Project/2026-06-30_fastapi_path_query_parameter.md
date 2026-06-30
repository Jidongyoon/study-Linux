# FastAPI 학습 기록 (2026-06-30)

## 학습 목표

- FastAPI 기본 라우팅 이해
- Path Parameter 개념 학습
- Type Hint 개념 이해
- HTTP 상태 코드 확인
- 예외 처리(HTTPException)
- Query Parameter 개념 학습

---

# 1. FastAPI 실행

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

- `--reload`
  - 코드 변경 시 자동으로 서버 재시작

---

# 2. GET API 작성

```python
@app.get("/")
def root():
    return {"message": "Hello LG Twins!"}
```

학습 내용

- FastAPI 라우팅
- JSON 응답
- 브라우저에서 API 확인

---

# 3. Path Parameter

```python
@app.get("/players/{player_id}")
def get_player(player_id: int):
```

## 학습 내용

Path Parameter는 URL의 일부를 변수처럼 사용하는 기능이다.

예시

```
/players/1
/players/2
/players/3
```

FastAPI는 URL 값을 자동으로 함수의 매개변수로 전달한다.

```
player_id = 1
```

---

# 4. Type Hint

```python
player_id: int
```

학습 내용

- URL 값은 기본적으로 문자열(String)
- Type Hint를 사용하면 FastAPI가 자동으로 정수(Integer)로 변환
- 잘못된 타입 입력 시 자동 검증

실습

```
/players/abc
```

결과

```
422 Unprocessable Entity
```

---

# 5. Dictionary 조회

```python
players = {
    1: {"name": "홍창기", "position": "OF"},
    2: {"name": "오스틴", "position": "1B"},
    3: {"name": "박동원", "position": "C"},
}
```

Path Parameter를 이용하여 선수 정보를 조회하는 API를 구현하였다.

---

# 6. 예외 처리 (HTTPException)

```python
raise HTTPException(
    status_code=404,
    detail="선수를 찾을 수 없습니다."
)
```

실습

```
/players/10
```

결과

```
404 Not Found
```

학습 내용

- 존재하지 않는 데이터를 요청하면 서버 오류(500)가 아니라 404를 반환하도록 처리

---

# 7. HTTP 상태 코드

| Status Code | 의미 |
|-------------|------|
| 200 | 정상 응답 |
| 404 | 요청한 데이터 없음 |
| 422 | 입력값 검증 실패 |

---

# 8. Query Parameter

```python
@app.get("/players")
def players(position: str = None):
```

예시

```
/players?position=OF
```

학습 내용

- Query Parameter는 검색 조건이나 옵션을 전달하는 기능
- URL의 `?` 뒤에 작성
- FastAPI가 자동으로 함수의 매개변수에 전달

예시

```
position = "OF"
```

현재는 Query Parameter를 정상적으로 전달받는 것까지 구현하였다.

---

# Path Parameter vs Query Parameter

| Path Parameter | Query Parameter |
|----------------|----------------|
| `/players/1` | `/players?position=OF` |
| 특정 대상 조회 | 검색 조건 전달 |
| URL 경로 사용 | URL ? 뒤 사용 |

---

# 오늘 배운 핵심

- FastAPI 실행
- GET API 작성
- Routing 이해
- Path Parameter
- Type Hint
- HTTPException
- HTTP 상태 코드
- Query Parameter
- FastAPI 자동 입력 검증
- API 테스트 및 로그 확인

---

# 느낀 점

오늘은 FastAPI의 핵심 개념인 Path Parameter와 Query Parameter의 차이를 이해하고, HTTPException을 활용한 예외 처리와 HTTP 상태 코드(200, 404, 422)의 동작을 직접 실습했다. 특히 FastAPI가 타입 힌트를 이용해 입력값을 자동 검증하는 과정을 확인하면서 API가 요청을 처리하는 흐름을 이해할 수 있었다.
# 📚 TIL - LLM Serving 기초와 추론 성능 이해

## 📌 오늘의 학습 주제

오늘은 **GPU 기반 LLM Serving이 어떻게 동작하는지**와 실제 서비스를 운영할 때 중요하게 보는 성능 지표에 대해 학습했다.

특히 다음 내용을 중심으로 공부했다.

- LLM의 Token 생성 방식
- Prefill과 Decode
- TTFT와 TPOT
- KV Cache
- Batching과 Throughput
- Tensor와 Attention의 기본 개념
- FastAPI를 이용한 API 기초

---

# 1. LLM은 어떻게 문장을 생성할까?

LLM은 완성된 문장을 한 번에 생성하는 것이 아니다.

입력된 Prompt를 처리한 뒤 **다음 Token을 하나 예측하고, 생성된 Token을 이용해 다시 다음 Token을 예측하는 과정을 반복**한다.

예를 들어

```text
입력

"오늘 점심은"

        ↓

"오늘 점심은 김치찌개"

        ↓

"오늘 점심은 김치찌개를"

        ↓

"오늘 점심은 김치찌개를 먹었다"
```

와 같은 방식으로 Token을 하나씩 생성한다.

즉, LLM의 기본적인 생성 과정은 다음과 같이 이해할 수 있다.

```text
Prompt 입력
    ↓
입력 처리
    ↓
다음 Token 예측
    ↓
Token 생성
    ↓
다음 Token 예측
    ↓
Token 생성
    ↓
반복...
```

이러한 특성 때문에 출력해야 하는 Token이 많아질수록 반복적인 추론 연산도 증가한다.

---

# 2. Prefill과 Decode

LLM 추론 과정은 크게 **Prefill과 Decode** 두 단계로 나누어 생각할 수 있다.

## Prefill

Prefill은 사용자가 입력한 **Prompt를 처음 처리하는 단계**이다.

```text
"쿠버네티스가 무엇인지 설명해줘"

              ↓

           Prefill
```

입력된 여러 Token을 처리하면서 이후 Token 생성에 필요한 정보를 계산한다.

## Decode

Prefill 이후에는 실제 출력 Token을 하나씩 생성한다.

```text
Prefill
   ↓
Token 1
   ↓
Token 2
   ↓
Token 3
   ↓
Token 4
   ↓
...
```

이 과정을 **Decode**라고 한다.

정리하면:

```text
사용자 Prompt
      ↓
   Prefill
      ↓
첫 번째 Token
      ↓
   Decode
      ↓
Token → Token → Token → ...
```

---

# 3. TTFT와 TPOT

LLM Serving에서는 단순히 전체 응답 시간만 측정하는 것이 아니라 **Token이 생성되는 과정 자체를 여러 성능 지표로 측정**한다.

## TTFT (Time To First Token)

사용자가 요청을 보낸 순간부터 **첫 번째 Token이 생성될 때까지 걸리는 시간**이다.

```text
사용자 요청
    │
    │
    │  ← TTFT →
    │
    ▼
첫 번째 Token
```

TTFT가 짧을수록 사용자는 서비스가 빠르게 반응한다고 느낄 수 있다.

---

## TPOT (Time Per Output Token)

첫 번째 Token이 생성된 이후 **출력 Token 하나를 생성하는 데 걸리는 평균 시간**이다.

```text
첫 Token
   │
   │ ← TPOT →
   ▼
두 번째 Token
   │
   │ ← TPOT →
   ▼
세 번째 Token
```

따라서 간단하게 생각하면

```text
TTFT
= 첫 응답이 얼마나 빨리 시작되는가?

TPOT
= 이후 답변이 얼마나 빠르게 생성되는가?
```

라고 이해할 수 있다.

---

# 4. KV Cache

LLM은 새로운 Token을 생성할 때 **이전에 입력되거나 생성된 Token의 정보를 계속 참고**한다.

만약 Token을 하나 생성할 때마다 이전 Token에 대한 Attention 계산을 모두 처음부터 다시 수행한다면 불필요한 연산이 계속 반복될 수 있다.

이를 줄이기 위해 사용하는 것이 **KV Cache**이다.

Attention 계산 과정에서 이전 Token의 **Key와 Value**를 저장해두고 다음 Token을 생성할 때 재사용한다.

```text
Token 처리
    ↓
Key / Value 계산
    ↓
KV Cache에 저장
    ↓
다음 Token 생성
    ↓
기존 Key / Value 재사용
```

따라서 KV Cache의 핵심은

> 이미 계산한 Key와 Value를 저장해두고 다음 Token 생성에서 다시 사용하여 반복 계산을 줄이는 것

이라고 이해했다.

LLM의 Context가 길어지거나 동시에 처리하는 요청이 많아지면 저장해야 할 KV Cache도 증가하기 때문에 **메모리 사용량과도 밀접한 관계가 있다.**

---

# 5. Batching과 Throughput

실제 LLM 서비스에서는 한 명의 사용자만 요청하는 것이 아니라 여러 사용자의 요청이 동시에 들어온다.

예를 들어

```text
사용자 A ─┐
사용자 B ─┤
사용자 C ─┼──→ LLM Server → GPU
사용자 D ─┘
```

GPU는 많은 연산을 병렬로 처리하는 데 강점이 있기 때문에 여러 요청을 묶어서 처리하면 GPU를 더욱 효율적으로 활용할 수 있다.

이를 **Batching**이라고 한다.

```text
Request A ─┐
Request B ─┼──→ Batch → GPU
Request C ─┤
Request D ─┘
```

여기서 중요한 개념이 **Throughput(처리량)**이다.

Throughput은 일정 시간 동안 얼마나 많은 요청이나 Token을 처리할 수 있는지를 나타낸다.

즉 LLM Serving에서는

```text
Latency
→ 한 요청을 얼마나 빠르게 처리하는가?

Throughput
→ 전체적으로 얼마나 많은 작업을 처리할 수 있는가?
```

를 함께 고려해야 한다.

Batching은 GPU를 효율적으로 사용하여 **전체 처리량을 높이는 데 중요한 역할**을 한다.

---

# 6. Tensor란?

AI 모델 내부에서는 문자 자체를 직접 계산하는 것이 아니라 **숫자로 변환하여 계산**한다.

예를 들어 하나의 Token도 모델 내부에서는 여러 숫자로 표현될 수 있다.

```text
Token
  ↓
숫자로 표현
  ↓
[0.12, 0.83, 0.41, ...]
```

이처럼 AI 연산에서 사용되는 **다차원 숫자 데이터 구조를 Tensor**라고 한다.

간단하게는

> Tensor = AI 모델이 계산하기 위해 사용하는 숫자 데이터 구조

라고 이해했다.

GPU에서는 이러한 Tensor를 대상으로 대량의 연산을 수행한다.

---

# 7. Attention 기초

LLM은 문장을 처리할 때 모든 Token의 정보를 똑같이 사용하는 것이 아니라 **현재 Token과 다른 Token 사이의 관련성을 계산**한다.

예를 들어

```text
"철수는 배가 고팠다. 그는 밥을 먹었다."
```

라는 문장에서 `그는`이라는 표현을 이해하기 위해서는 앞에 등장했던 `철수`라는 정보가 중요하다.

Attention은 이러한 관계를 계산하여 **어떤 Token의 정보를 더 중요하게 참고할 것인지 결정하는 과정**이라고 이해했다.

Attention에서는 다음과 같은 개념이 등장한다.

```text
Query
→ 어떤 정보를 찾을 것인가?

Key
→ 어떤 정보와 관련 있는지 비교

Value
→ 실제로 가져와서 사용할 정보

Attention Weight
→ 해당 정보를 얼마나 중요하게 반영할 것인가?
```

전체 흐름을 단순화하면

```text
Query와 Key 비교
       ↓
Token 사이의 관련성 계산
       ↓
Attention Weight
       ↓
Weight × Value
       ↓
Weighted Sum
       ↓
필요한 정보들을 종합
```

과 같은 방식으로 이해할 수 있다.

아직 실제 수학적인 계산 과정은 어렵게 느껴지기 때문에 우선 **Attention이 왜 필요한지와 Q, K, V가 어떤 역할을 하는지**를 중심으로 이해했다.

---

# 8. FastAPI와 Pydantic 기초

LLM Serving을 공부하면서 실제 모델을 서비스로 제공하려면 **API가 필요하다**는 점도 함께 공부했다.

FastAPI를 이용하면 Python으로 간단하게 API 서버를 만들 수 있다.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "내 첫 FastAPI 서버"}
```

Uvicorn으로 서버를 실행하면

```bash
uvicorn main:app --reload --port 8000
```

다음과 같은 흐름으로 요청을 처리할 수 있다.

```text
Client
   ↓
HTTP Request
   ↓
Uvicorn
   ↓
FastAPI
   ↓
Python 함수 실행
   ↓
HTTP Response
```

POST 요청에서는 Pydantic의 `BaseModel`을 이용해 클라이언트가 보내는 데이터의 형태를 정의할 수 있다.

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
```

이는

```text
User라는 데이터에는
name이 필요하고

name의 데이터 타입은
문자열(str)이어야 한다.
```

라는 설계도라고 이해했다.

Pydantic은 들어온 데이터가 이 규칙에 맞는지 확인하는 **유효성 검사(Validation)**도 수행한다.

---

# 🔗 오늘 배운 내용 연결하기

오늘 배운 내용을 전체적으로 연결하면 다음과 같다.

```text
사용자
  │
  │ HTTP Request
  ▼
FastAPI
  │
  │ Prompt 전달
  ▼
LLM Serving
  │
  ▼
Prefill
  │
  │ Attention
  │ Tensor 연산
  │ KV Cache 생성
  ▼
첫 Token 생성
  │
  │ ← TTFT
  ▼
Decode
  │
  │ KV Cache 재사용
  │
  │ ← TPOT
  ▼
Token
  ↓
Token
  ↓
Token
  ↓
최종 응답
  │
  ▼
FastAPI
  │
  ▼
사용자
```

여러 사용자의 요청이 들어오는 실제 서비스 환경에서는 여기에 **Batching**을 적용하여 GPU의 처리량을 높일 수 있다.

---

# ✍️ 오늘 배운 점

오늘은 단순히 LLM 모델을 실행하는 방법보다 **LLM Serving 내부에서 실제로 어떤 과정이 일어나는지**를 중심으로 공부했다.

특히 다음 흐름을 이해한 것이 가장 중요했다.

```text
Prompt
 ↓
Prefill
 ↓
TTFT
 ↓
첫 Token
 ↓
Decode
 ↓
KV Cache 활용
 ↓
TPOT
 ↓
Token 반복 생성
```

이전에는 LLM에게 질문을 보내면 GPU가 답변을 한 번에 계산한다고 막연하게 생각했지만, 실제로는 **Prompt를 먼저 처리하고 Token을 하나씩 반복적으로 생성하며, KV Cache를 이용해 이전 계산 결과를 재사용한다는 것**을 알게 되었다.

또한 실제 LLM 서비스를 운영할 때는 단순히 모델이 실행되는지만 보는 것이 아니라 **TTFT, TPOT, Latency, Throughput, Batching 등 여러 성능 요소를 함께 고려해야 한다는 점**을 배웠다.

Attention의 Query, Key, Value와 같은 내부 계산은 아직 어렵게 느껴지지만, 먼저 전체적인 LLM Serving 흐름을 이해한 뒤 하나씩 깊게 공부해 나갈 예정이다.
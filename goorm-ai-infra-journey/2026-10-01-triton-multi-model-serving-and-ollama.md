# 오늘의 배움일기

## 학습 주제

**Triton Inference Server를 이용한 여러 모델 서빙과 Ollama 모델 관리**

오늘 수업에서는 하나의 AI 모델만 실행하는 것에서 더 나아가, **여러 모델을 하나의 서버에서 관리하고 요청을 처리하는 방법**을 학습했다.

Kaggle에서는 **Triton Inference Server**를 이용해 여러 모델을 하나의 서버에 올려 보았고, Mac에서는 **Ollama**를 이용해 여러 LLM이 메모리에 어떻게 올라가고 내려가는지 확인했다.

또한 `config.pbtxt` 설정과 `dynamic_batching`을 통해 여러 요청을 효율적으로 처리하는 방법도 실습했다.

---

## 1. Triton Inference Server

Triton은 AI 모델을 서버 형태로 실행하고 외부에서 HTTP 요청을 받아 추론 결과를 반환할 수 있게 해주는 **Inference Server**이다.

오늘 실습에서는 다음과 같은 구조로 모델을 실행했다.

```text
Client
   │
   │ HTTP Request
   ▼
Triton Inference Server
   │
   ├── sentiment
   │
   └── spam
```

하나의 Triton 서버 안에서 여러 모델을 관리할 수 있다는 것이 중요한 특징이다.

예를 들어 `sentiment` 모델과 `spam` 모델을 하나의 Triton 서버에 올리고 각각 다른 주소로 요청을 보낼 수 있다.

```text
/v2/models/sentiment/infer

/v2/models/spam/infer
```

---

## 2. Model Repository

Triton은 아무 위치에 있는 모델을 자동으로 사용하는 것이 아니라 **Model Repository**라는 정해진 구조로 모델을 저장한다.

오늘 실습에서는 다음과 같은 형태로 모델을 구성했다.

```text
models/
│
├── sentiment/
│   ├── config.pbtxt
│   └── 1/
│       └── model.py
│
└── spam/
    ├── config.pbtxt
    └── 1/
        └── model.py
```

기본적인 구조는 다음과 같다.

```text
models/
   ↓
모델 이름/
   ↓
버전 번호/
   ↓
모델 파일
```

예를 들어

```text
sentiment/1/model.py
```

에서

- `sentiment` → 모델 이름
- `1` → 모델 버전
- `model.py` → 실제 모델 코드

를 의미한다.

Triton 서버를 실행하면 이 Model Repository를 읽어서 사용할 모델을 등록한다.

---

## 3. Triton 서버 실행

오늘 Kaggle에서는 다음과 같은 방식으로 Triton 서버를 실행했다.

```bash
nohup ./tritonserver --model-repository=models > triton.log 2>&1 &
```

여기서

```text
./tritonserver
→ Triton 서버 실행

--model-repository=models
→ models 폴더를 Model Repository로 사용

> triton.log
→ 서버 로그를 triton.log 파일에 저장

2>&1
→ 에러 로그도 같은 파일에 저장

&
→ 백그라운드 실행
```

서버가 정상적으로 모델을 불러오면 모델 상태가 `READY`로 표시된다.

```text
sentiment   READY
spam        READY
```

즉,

```text
Model Repository
      ↓
Triton Server 실행
      ↓
모델 설정 확인
      ↓
모델 Load
      ↓
READY
      ↓
추론 요청 가능
```

의 흐름으로 동작한다.

---

## 4. config.pbtxt

Triton에서는 모델마다 `config.pbtxt`라는 설정 파일을 사용할 수 있다.

모델 파일이 **실제로 어떤 작업을 수행할지** 정의한다면,

`config.pbtxt`는 **Triton이 그 모델을 어떻게 실행하고 관리할지** 알려주는 설정 파일이라고 이해했다.

오늘 살펴본 주요 설정은 다음과 같다.

```text
name
max_batch_size
input / output
instance_group
dynamic_batching
```

각각의 역할을 간단하게 정리하면 다음과 같다.

```text
name
→ 모델 이름

max_batch_size
→ 한 번에 처리할 수 있는 최대 Batch 크기

input
→ 모델이 받을 입력 데이터

output
→ 모델이 반환할 출력 데이터

instance_group
→ 모델 실행 인스턴스 설정

dynamic_batching
→ 여러 요청을 묶어서 실행하도록 설정
```

특히 모델 이름은 Model Repository의 폴더 구조와 맞아야 한다.

---

## 5. Dynamic Batching

오늘 수업에서 중요하게 실습한 기능 중 하나가 **Dynamic Batching**이다.

일반적으로 요청이 여러 개 들어오면 각각 실행할 수도 있다.

```text
요청 1 → 실행
요청 2 → 실행
요청 3 → 실행
요청 4 → 실행
...
요청 8 → 실행
```

이 경우 요청 8개에 대해 모델 실행도 여러 번 발생할 수 있다.

Dynamic Batching을 사용하면 짧은 시간 동안 들어온 요청들을 모아서 **하나의 Batch로 묶어 처리**할 수 있다.

```text
요청 1 ─┐
요청 2 ─┤
요청 3 ─┤
요청 4 ─┤
요청 5 ─┤
요청 6 ─┤
요청 7 ─┤
요청 8 ─┘
         ↓
      Batch
         ↓
     모델 실행
```

오늘 실습에서는 요청 8개가 들어왔을 때 Dynamic Batching 설정에 따라 **여러 요청이 묶여 실행되는 것**을 확인했다.

Dynamic Batching을 제거했을 때와 비교하면서 모델의 실행 횟수가 달라지는 것도 확인했다.

즉,

```text
Dynamic Batching

여러 Request
      ↓
잠시 모음
      ↓
Batch 생성
      ↓
한 번에 모델 실행
```

이라고 이해할 수 있다.

---

## 6. 두 번째 모델 추가하기

처음에는 `sentiment` 모델 하나를 사용했지만 이후 `spam` 모델을 추가했다.

```text
Triton Server
      │
      ├── sentiment
      │
      └── spam
```

새로운 모델을 Model Repository에 추가한 후 서버를 다시 실행하면 Triton이 저장소를 다시 읽고 두 모델을 Load한다.

```bash
pkill -f "[t]ritonserver"
```

서버를 종료한 뒤 다시 실행했다.

```bash
nohup ./tritonserver --model-repository=models > triton.log 2>&1 &
```

정상적으로 올라왔다면 두 모델 모두 `READY` 상태가 된다.

```text
sentiment    READY
spam         READY
```

이를 통해 **하나의 Triton 서버에서 여러 모델을 동시에 제공할 수 있다는 것**을 확인했다.

---

## 7. 모델별 API 요청

Triton에서는 요청할 모델의 이름을 URL에 지정할 수 있다.

sentiment 모델은

```text
/v2/models/sentiment/infer
```

spam 모델은

```text
/v2/models/spam/infer
```

처럼 사용한다.

예를 들어 spam 모델에 다음과 같은 문장을 전달할 수 있다.

```text
무료 쿠폰 당첨! 지금 클릭하세요
```

요청의 전체적인 흐름은 다음과 같다.

```text
사용자
   ↓
HTTP Request
   ↓
/v2/models/spam/infer
   ↓
Triton Server
   ↓
spam 모델
   ↓
추론
   ↓
결과 반환
```

하나의 서버를 사용하지만 **URL의 모델 이름을 통해 어떤 모델에게 요청을 전달할지 구분**할 수 있다.

---

## 8. 설정이 잘못되면 서버가 뜨지 않을 수 있다

Triton에서는 Model Repository의 구조나 모델 설정이 중요하다.

모델의 이름이나 설정이 잘못되어 있으면 정상적으로 모델을 Load하지 못할 수 있고, 실습에서는 **모델 하나의 설정 문제로 서버 전체가 정상적으로 뜨지 않는 상황**도 확인했다.

따라서 Triton 서버가 정상적으로 실행되지 않는다면 먼저 로그를 확인해야 한다.

```bash
grep -E "READY|UNAVAILABLE" triton.log
```

또는 `triton.log`의 오류 메시지를 확인한다.

```text
서버 실행 실패
      ↓
triton.log 확인
      ↓
어떤 모델에서 오류가 발생했는지 확인
      ↓
Model Repository / config.pbtxt 확인
      ↓
수정
      ↓
서버 재실행
```

이 과정을 통해 AI 모델을 운영할 때도 **로그를 통한 문제 확인이 중요하다**는 것을 알게 되었다.

---

## 9. Ollama의 모델 관리 방식

오후에는 Mac에서 Ollama를 이용하여 LLM 모델이 메모리에 어떻게 올라가고 내려가는지 확인했다.

현재 메모리에 올라가 있는 모델은 다음 명령어로 확인할 수 있다.

```bash
ollama ps
```

모델을 실행하면 해당 모델이 메모리에 올라간다.

```bash
ollama run qwen2.5:0.5b "안녕"
```

다른 모델도 실행할 수 있다.

```bash
ollama run gemma3:270m "안녕"
```

이후

```bash
ollama ps
```

를 실행하면 현재 올라가 있는 모델을 확인할 수 있다.

전체 흐름은 다음과 같다.

```text
ollama run
     ↓
모델 Load
     ↓
메모리에 모델 올라감
     ↓
추론
     ↓
일정 시간 사용하지 않음
     ↓
모델이 내려갈 수 있음
```

---

## 10. Ollama에서 모델 내리기

현재 올라가 있는 모델은 직접 내릴 수도 있다.

```bash
ollama stop gemma3:270m
```

모델을 내린 뒤 다시 요청하면 모델을 다시 메모리에 Load해야 한다.

따라서 처음 요청할 때는 **모델을 불러오는 시간(load duration)**이 추가될 수 있다.

```text
모델이 이미 올라가 있음

Request
   ↓
Inference
   ↓
Response
```

반대로 모델이 내려가 있다면

```text
모델이 내려가 있음

Request
   ↓
Model Load
   ↓
Inference
   ↓
Response
```

과정이 필요하다.

따라서 내려간 모델의 첫 요청은 이미 올라가 있는 모델보다 느려질 수 있다.

---

## 11. Ollama에서 동시에 올릴 모델 수 제한

실습에서는 Ollama 서버를 직접 실행하면서 동시에 메모리에 올릴 모델 수를 제한하는 설정도 사용했다.

먼저 기존 Ollama 서비스를 중지했다.

```bash
brew services stop ollama
```

그리고 다음과 같이 실행했다.

```bash
OLLAMA_MAX_LOADED_MODELS=1 ollama serve
```

이 설정은 동시에 Load해 둘 모델의 수를 `1`로 제한한다.

즉,

```text
Model A 실행
   ↓
Model A 메모리에 Load

Model B 실행
   ↓
동시에 하나만 유지
   ↓
필요에 따라 모델 교체
```

와 같은 상황을 관찰할 수 있다.

실습이 끝난 후에는 직접 실행한 Ollama를 종료하고 기존 서비스를 다시 시작했다.

```bash
brew services start ollama
```

---

## 12. vLLM, Triton, Ollama 비교

이전 수업에서 사용했던 vLLM과 오늘 사용한 Triton, Ollama가 각각 어떤 상황에서 사용되는지도 정리했다.

```text
vLLM
→ LLM 서빙에 특화
→ 하나의 LLM에 많은 요청을 처리하는 상황

Triton
→ 여러 AI 모델을 하나의 서버에서 관리
→ Model Repository와 config.pbtxt 사용
→ Dynamic Batching 지원

Ollama
→ 로컬에서 LLM을 간편하게 실행
→ 모델을 쉽게 올리고 내릴 수 있음
→ ollama ps로 현재 Load된 모델 확인
```

오늘 수업에서는 이를 다음과 같은 관점으로 이해했다.

```text
LLM 하나를 많이 돌리기
        ↓
       vLLM

작은 모델 여러 개를 한 서버에서 관리
        ↓
      Triton

Mac에서 LLM을 간편하게 실행
        ↓
      Ollama
```

---

## ✍️ 오늘 배운 점

오늘 수업을 통해 AI 모델을 실행하는 것뿐만 아니라 **여러 모델을 실제 서버에서 어떻게 관리하고 요청을 효율적으로 처리하는지** 배울 수 있었다.

특히 Triton에서는 단순히 모델 파일만 있으면 되는 것이 아니라,

```text
Model Repository
      ↓
모델별 폴더
      ↓
버전 폴더
      ↓
모델 파일
      +
config.pbtxt
      ↓
Triton Server
```

와 같은 구조가 필요하다는 것을 알게 되었다.

또한 `dynamic_batching`을 이용하면 여러 요청을 Batch로 묶어 모델 실행을 효율적으로 처리할 수 있다는 것도 직접 확인했다.

Ollama 실습에서는 모델이 항상 메모리에 올라가 있는 것이 아니라 **필요할 때 Load되고 사용하지 않으면 내려갈 수 있으며**, 다시 올라올 때는 `load duration` 때문에 첫 요청이 느려질 수 있다는 것도 확인했다.

오늘 내용을 전체적으로 정리하면 다음과 같다.

```text
여러 AI 모델
     ↓
Model Repository 구성
     ↓
config.pbtxt 설정
     ↓
Triton Server 실행
     ↓
여러 모델 READY
     ↓
HTTP 요청
     ↓
Dynamic Batching
     ↓
모델 추론

그리고

Ollama
     ↓
모델 호출
     ↓
메모리에 Load
     ↓
추론
     ↓
사용하지 않으면 내려감
     ↓
다시 호출하면 Load 후 추론
```

어제는 **GPU와 VRAM, vLLM, Kubernetes GPU 자원 관리**를 중심으로 공부했다면, 오늘은 한 단계 더 나아가 **실제 추론 서버에서 여러 AI 모델을 어떻게 구성하고 운영하며 요청을 효율적으로 처리하는지** 이해할 수 있었다.
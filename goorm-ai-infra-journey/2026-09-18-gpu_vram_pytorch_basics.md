# 📚 GPU 기초와 PyTorch를 이용한 LLM 실행

## 📌 오늘의 학습 주제

오늘은 AI 모델을 운영하기 위해 필요한 **GPU와 VRAM의 기본 개념**을 학습하고, PyTorch를 이용해 실제 GPU 메모리를 확인하고 AI 모델을 GPU에 올려 실행해보았다.

주요 학습 내용은 다음과 같다.

- CPU와 GPU의 차이
- GPU Core와 병렬 연산
- `nvidia-smi`를 이용한 GPU 상태 확인
- CUDA와 GPU 선택
- VRAM의 역할
- PyTorch의 GPU 메모리 관리
- `allocated`와 `reserved`의 차이
- 모델 Parameter와 Weight
- FP16 / FP32와 모델 크기
- 모델을 GPU에 올리는 과정
- 실제 모델의 VRAM 사용량 확인
- LLM의 Token 생성 방식

---

# 1. CPU와 GPU의 차이

CPU와 GPU는 모두 계산을 수행하지만 구조와 목적에 차이가 있다.

CPU는 비교적 적은 수의 강력한 Core를 이용하여 다양한 작업을 처리하는 데 적합하다.

반면 GPU는 많은 수의 Core를 가지고 있으며, **비슷한 계산을 동시에 처리하는 병렬 연산**에 강하다.

AI 모델에서는 매우 많은 숫자를 대상으로 곱셈과 덧셈 같은 연산을 반복하기 때문에 GPU의 병렬 처리 능력이 중요하다.

```text
CPU
Core ── 계산
Core ── 계산
Core ── 계산

→ 적은 수의 강력한 Core
→ 다양한 작업 처리에 강함


GPU
Core Core Core Core Core Core ...
Core Core Core Core Core Core ...
Core Core Core Core Core Core ...

→ 많은 수의 Core
→ 같은 종류의 대량 계산을 동시에 처리하는 데 강함
```

따라서 AI 모델의 대규모 연산에서는 GPU를 사용하는 것이 유리하다.

---

# 2. nvidia-smi

NVIDIA GPU를 사용하는 환경에서는 `nvidia-smi` 명령어를 이용하여 GPU의 상태를 확인할 수 있다.

```bash
nvidia-smi
```

이를 통해 다음과 같은 정보를 확인할 수 있다.

- GPU 모델
- GPU 사용률
- VRAM 전체 용량
- 현재 VRAM 사용량
- GPU를 사용하는 Process
- Driver / CUDA 관련 정보

Linux에서 CPU와 Memory 상태를 `top`이나 `htop`으로 확인하는 것처럼, GPU 환경에서는 `nvidia-smi`가 기본적인 상태 확인 도구로 사용된다.

특정 GPU의 메모리 사용량만 확인할 수도 있다.

```bash
nvidia-smi --query-gpu=memory.used --format=csv
```

예시:

```text
memory.used [MiB]
14336 MiB
```

---

# 3. CUDA와 GPU 선택

PyTorch에서는 CUDA를 이용하여 NVIDIA GPU에서 연산을 수행할 수 있다.

GPU가 사용 가능한지 확인하려면 다음과 같이 확인할 수 있다.

```python
import torch

print(torch.cuda.is_available())
print(torch.cuda.device_count())
```

예를 들어 GPU가 2개 있는 환경에서는 다음과 같이 출력될 수 있다.

```text
True
2
```

`CUDA_VISIBLE_DEVICES` 환경변수를 사용하면 프로그램에서 사용할 GPU를 제한할 수도 있다.

예를 들어 GPU 0번만 보이게 하려면:

```bash
CUDA_VISIBLE_DEVICES=0 python -c "import torch; print(torch.cuda.device_count())"
```

결과:

```text
1
```

즉,

```text
실제 서버

GPU 0
GPU 1

      ↓ CUDA_VISIBLE_DEVICES=0

프로그램

GPU 0만 보임
```

과 같은 구조가 된다.

이러한 GPU 자원 선택은 이후 여러 GPU를 사용하는 서버나 Container/Kubernetes 환경에서 GPU를 분배할 때도 중요한 개념이 된다.

---

# 4. VRAM이란?

GPU에는 GPU 전용 메모리인 **VRAM(Video RAM)**이 존재한다.

AI 모델을 GPU에서 실행하려면 모델의 가중치와 계산에 필요한 데이터가 VRAM에 올라가야 한다.

```text
Disk / Storage
      ↓
모델 파일
      ↓
RAM
      ↓
VRAM
      ↓
GPU 계산
```

예를 들어 NVIDIA T4의 VRAM이 약 14.7GiB라면 AI 모델과 계산에 필요한 데이터가 이 공간 안에 들어갈 수 있어야 한다.

따라서 AI 모델을 운영할 때는 GPU의 계산 성능뿐만 아니라 **VRAM 용량도 매우 중요한 자원**이다.

---

# 5. Tensor

PyTorch에서 자주 사용하는 기본적인 데이터 구조가 **Tensor**이다.

Tensor는 쉽게 생각하면 **숫자들을 여러 차원으로 모아놓은 데이터 구조**라고 볼 수 있다.

예를 들어:

```text
[1, 2, 3]
```

또는

```text
[
  [1, 2, 3],
  [4, 5, 6]
]
```

처럼 숫자가 배열된 형태를 표현할 수 있다.

AI에서는 입력 데이터, 중간 계산 결과, 모델의 Weight 등이 Tensor 형태로 처리된다.

PyTorch를 사용하면 Tensor를 CPU뿐만 아니라 GPU에 올려 계산할 수도 있다.

```python
import torch

a = torch.tensor([1, 2, 3])

a_gpu = a.to("cuda")
```

여기서

```python
.to("cuda")
```

는 해당 Tensor를 GPU에서 사용할 수 있도록 이동시키는 의미이다.

---

# 6. Parameter와 Weight

AI 모델에는 학습을 통해 결정된 매우 많은 숫자가 들어 있다.

이러한 학습 가능한 값들을 **Parameter(파라미터)**라고 하며, 대표적인 것이 **Weight(가중치)**이다.

예를 들어 다음과 같은 행렬이 있다고 생각할 수 있다.

```text
4096 × 4096
```

이 행렬 안에 들어가는 숫자의 개수는

```text
4096 × 4096
= 16,777,216
```

개이다.

이러한 값들이 모델의 Weight라면 이 행렬 하나만으로도 약 1,677만 개의 Parameter를 가지게 된다.

실제 LLM에서는 이런 구조가 여러 Layer에 반복되기 때문에 전체 Parameter 수가 수억~수백억 개까지 증가할 수 있다.

그래서 모델에서 다음과 같은 표현을 볼 수 있다.

```text
0.5B
1.5B
7B
70B
```

여기서 `B`는 Billion을 의미한다.

```text
0.5B ≈ 5억 Parameter
1.5B ≈ 15억 Parameter
7B   ≈ 70억 Parameter
70B  ≈ 700억 Parameter
```

---

# 7. 모델 크기와 VRAM 계산

모델을 GPU에 올리려면 가중치를 저장할 VRAM이 필요하다.

가중치가 차지하는 메모리는 기본적으로 다음과 같이 생각할 수 있다.

```text
가중치 VRAM ≈ Parameter 수 × Parameter 하나의 Byte
```

FP16은 하나의 값을 16bit로 표현한다.

```text
16 bit = 2 Byte
```

따라서 FP16 모델이라면 대략적으로 다음과 같이 계산할 수 있다.

```text
0.5B × 2 Byte ≈ 1 GB
1.5B × 2 Byte ≈ 3 GB
7B   × 2 Byte ≈ 14 GB
70B  × 2 Byte ≈ 140 GB
```

예를 들어 7B 모델을 FP16으로 사용하면 **가중치만 약 14GB**가 필요하다.

하지만 실제 추론에서는 가중치만 VRAM을 사용하는 것이 아니다.

```text
VRAM

├── Model Weight
├── KV Cache
├── Activation
└── 기타 연산 Overhead
```

따라서 14GB VRAM이 있다고 해서 가중치가 14GB인 모델을 그대로 안정적으로 실행할 수 있는 것은 아니다.

**모델의 가중치 크기 + 추론 과정에서 추가로 필요한 메모리**를 함께 고려해야 한다.

---

# 8. FP16과 FP32

모델의 숫자를 어떤 데이터 타입으로 저장하는지에 따라서도 필요한 메모리가 달라진다.

대표적으로 다음과 같은 차이가 있다.

| Data Type | 하나의 값 | 특징 |
|---|---:|---|
| FP32 | 4 Byte | 높은 정밀도, 메모리 사용량 큼 |
| FP16 | 2 Byte | FP32보다 메모리 사용량 감소 |

예를 들어 7B Parameter 모델이라면 단순 가중치 기준으로:

```text
FP32

7B × 4 Byte
≈ 28 GB


FP16

7B × 2 Byte
≈ 14 GB
```

가 된다.

따라서 같은 모델이라도 어떤 데이터 타입으로 모델을 올리는지에 따라 필요한 VRAM이 크게 달라질 수 있다.

---

# 9. PyTorch란?

**PyTorch는 AI/딥러닝 모델을 만들고 실행하기 위한 Python 기반 프레임워크(라이브러리)**이다.

Python 코드에서 다음과 같이 사용할 수 있다.

```python
import torch
```

여기서 `torch`가 PyTorch에서 사용하는 핵심 Python 모듈이다.

예를 들어:

```python
torch.cuda.is_available()
```

를 나누어 보면 다음과 같이 이해할 수 있다.

```text
torch
 ↓
PyTorch

torch.cuda
 ↓
PyTorch의 CUDA 관련 기능

torch.cuda.is_available()
 ↓
CUDA GPU를 사용할 수 있는지 확인하는 함수
```

`.`은 객체나 모듈 내부의 기능에 접근한다는 의미이고, `()`는 함수를 실행한다는 의미이다.

---

# 10. PyTorch의 VRAM 관리

PyTorch에서는 GPU 메모리 사용량을 확인할 수 있다.

현재 Tensor 등이 실제로 사용하고 있는 VRAM:

```python
torch.cuda.memory_allocated()
```

PyTorch가 확보해 놓은 VRAM:

```python
torch.cuda.memory_reserved()
```

정리하면:

```text
GPU 전체 VRAM

┌──────────────────────────────┐
│                              │
│  PyTorch Reserved            │
│  ┌───────────────────────┐   │
│  │ Allocated             │   │
│  │ 실제 사용 중          │   │
│  └───────────────────────┘   │
│                              │
│  남은 공간 = PyTorch Cache   │
│                              │
└──────────────────────────────┘
```

즉,

```text
allocated = 실제로 사용하는 양
reserved  = PyTorch가 확보해 둔 양
```

으로 이해할 수 있다.

---

# 11. 왜 Tensor를 삭제해도 nvidia-smi의 VRAM이 그대로일까?

PyTorch는 GPU 메모리를 한 번 사용한 뒤 바로 GPU에 반환하지 않을 수 있다.

이유는 **다음 GPU 연산에서 빠르게 다시 사용하기 위해서**이다.

예를 들어:

```python
del a_gpu
```

로 Tensor를 삭제하면 실제 사용 중인 메모리는 감소할 수 있다.

```python
torch.cuda.memory_allocated()
```

결과:

```text
0 GB
```

하지만:

```python
torch.cuda.memory_reserved()
```

에서는 여전히 많은 메모리가 잡혀 있을 수 있다.

```text
14 GB
```

그리고 `nvidia-smi`에서도 높은 GPU 메모리 사용량이 보일 수 있다.

```text
Tensor 삭제

allocated
14GB → 0GB

reserved
14GB → 14GB

nvidia-smi
약 14GB
```

즉, Tensor는 사라졌지만 PyTorch가 GPU 메모리를 **Cache 형태로 확보해 놓은 상태**이다.

---

# 12. torch.cuda.empty_cache()

PyTorch가 확보하고 있지만 현재 사용하지 않는 GPU 메모리를 반환하려면 다음을 사용할 수 있다.

```python
torch.cuda.empty_cache()
```

이후 `nvidia-smi`를 확인하면 GPU 메모리 사용량이 감소할 수 있다.

```text
Tensor 사용 중

allocated : 14GB
reserved  : 14GB

        ↓ del

allocated : 0GB
reserved  : 14GB

        ↓ empty_cache()

allocated : 0GB
reserved  : 감소

        ↓

nvidia-smi 사용량 감소
```

하지만 `empty_cache()`는 평소에 계속 호출하는 명령이라기보다는, **PyTorch가 확보한 미사용 캐시를 다른 프로세스 등에 양보할 필요가 있을 때 사용할 수 있다.**

또한 현재 실제로 사용 중인 Tensor의 메모리를 강제로 제거하는 기능은 아니다.

---

# 13. allocated / reserved / nvidia-smi의 차이

오늘 실습에서 중요한 부분 중 하나였다.

```text
memory_allocated()
        ↓
PyTorch Tensor 등이
현재 실제 사용하는 메모리


memory_reserved()
        ↓
PyTorch가 CUDA로부터
확보해 놓은 메모리


nvidia-smi
        ↓
GPU/Driver 관점에서
프로세스가 점유하고 있는 GPU 메모리
```

따라서 `nvidia-smi`에서 GPU 메모리가 많이 사용되고 있다고 표시된다고 해서 그 전체가 현재 Tensor 계산에 직접 사용되고 있다고 단정하면 안 된다.

AI 서비스를 운영할 때는 **GPU 전체 상태와 프레임워크 내부의 메모리 상태를 함께 확인하는 것이 중요하다.**

---

# 14. 실제 모델을 GPU에 올리기

Hugging Face의 모델을 PyTorch 기반으로 불러와 GPU에 올리는 과정도 실습했다.

예:

```python
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-0.5B-Instruct",
    torch_dtype=torch.float16,
).to("cuda")
```

여기서 중요한 부분을 나누어 보면:

```text
from_pretrained()
        ↓
학습이 완료된 모델과 가중치를 불러옴


torch_dtype=torch.float16
        ↓
가중치를 FP16 형태로 사용


.to("cuda")
        ↓
모델을 GPU에서 사용할 수 있도록 이동
```

따라서 **모델을 GPU에 올린다**는 것은 단순히 GPU 기능을 켠다는 의미가 아니다.

모델의 Weight 등을 GPU가 계산할 수 있도록 **VRAM에 적재하는 과정**이라고 이해할 수 있다.

```text
Hugging Face / Disk
        ↓
Model Weight
        ↓
PyTorch
        ↓
.to("cuda")
        ↓
VRAM
        ↓
GPU 연산
```

---

# 15. 모델 로딩 전후 VRAM 비교

모델을 올리기 전의 GPU 메모리를 확인하고:

```python
before = torch.cuda.memory_allocated()
```

모델을 GPU에 올린 뒤:

```python
after = torch.cuda.memory_allocated()
```

두 값을 비교하면 실제 모델이 어느 정도의 VRAM을 사용했는지 확인할 수 있다.

오늘 실습에서는 약 0.49B 규모 모델을 FP16으로 올렸을 때 약 0.92GiB의 VRAM 증가량을 확인했다.

이론적으로:

```text
0.49B × 2 Byte
≈ 약 0.98GB
```

정도의 가중치 크기를 예상할 수 있다.

실제 측정 결과와 비슷한 수준이 나오는 것을 통해

```text
Parameter 수 × Byte
        ↓
예상 모델 Weight 크기
        ↓
실제 VRAM 사용량 측정
```

이라는 관계를 확인할 수 있었다.

---

# 16. LLM이 문장을 처리하는 과정

LLM은 우리가 입력한 문자열을 그대로 계산하는 것이 아니다.

먼저 **Tokenizer**가 문자열을 Token ID라는 숫자로 변환한다.

```text
사용자 문장

"GPU와 CPU의 차이를 알려줘"

        ↓

Tokenizer

        ↓

Token ID

[1234, 582, 91,  ...]

        ↓

LLM
```

모델은 이러한 숫자들을 이용하여 계산한다.

그리고 모델이 생성한 Token ID는 다시 Tokenizer를 통해 사람이 읽을 수 있는 문자열로 변환된다.

```text
문장
 ↓
Tokenizer
 ↓
Token ID
 ↓
LLM
 ↓
Token ID
 ↓
Tokenizer
 ↓
문장
```

---

# 17. LLM은 답변을 한 번에 만들지 않는다

LLM의 중요한 특징은 **완성된 문장을 한 번에 생성하지 않는다는 것**이다.

모델은 현재까지의 Token을 이용하여 **다음 Token 하나를 예측한다.**

```text
입력

"GPU는"

        ↓

다음 Token 예측

"병렬"

        ↓

현재 문장

"GPU는 병렬"

        ↓

다음 Token 예측

"연산"

        ↓

"GPU는 병렬 연산"

        ↓

다음 Token 예측...

        ↓

반복
```

즉, 답변이 60 Token이라면 다음 Token을 생성하는 과정이 여러 번 반복된다.

코드에서는 다음과 같이 생성 길이를 제한할 수 있다.

```python
out = model.generate(
    **inputs,
    max_new_tokens=60
)
```

`max_new_tokens=60`은 **새롭게 생성할 Token의 최대 개수를 60개로 제한한다는 의미**이다.

---

# 18. LLM 서비스가 GPU 자원을 많이 사용하는 이유

일반적인 웹 API는 요청 하나에 필요한 로직을 실행하고 결과를 반환하는 경우가 많다.

하지만 LLM은 답변을 생성하면서 다음 Token 계산을 계속 반복한다.

```text
사용자 요청
    ↓
Prompt Token 처리
    ↓
Token 생성
    ↓
Token 생성
    ↓
Token 생성
    ↓
Token 생성
    ↓
...
    ↓
응답 완료
```

따라서 생성해야 할 Token이 많아질수록:

- GPU 계산량 증가
- 응답 시간 증가
- KV Cache 등 메모리 사용량 증가 가능
- 동시에 많은 요청이 들어오면 GPU 부하 증가

등을 고려해야 한다.

이 때문에 LLM Serving에서는 단순히 **모델이 실행되는가?**만 확인하는 것이 아니라,

```text
GPU 사용률
VRAM 사용량
모델 크기
입력/출력 Token 수
동시 요청
Latency
Throughput
```

등을 함께 고려해야 한다.

---

# 19. 오늘 배운 전체 흐름

오늘 배운 내용을 AI 인프라 관점에서 연결하면 다음과 같다.

```text
AI Model
   │
   │ 모델의 Weight 존재
   ▼
Model Size 확인
   │
   │ Parameter × Byte
   ▼
필요한 VRAM 예상
   │
   ▼
GPU 선택
   │
   │ CUDA
   ▼
PyTorch
   │
   │ .to("cuda")
   ▼
VRAM에 Model 적재
   │
   ▼
GPU에서 추론
   │
   ├── Token 생성 반복
   ├── VRAM 사용
   └── GPU 연산
   │
   ▼
nvidia-smi
memory_allocated()
memory_reserved()
   │
   ▼
GPU 자원 상태 확인
```

---

# ✍️ 오늘 수업을 통해 배운 점

오늘 수업에서는 단순히 GPU가 AI 연산에 사용된다는 사실보다 **AI 모델이 실제로 GPU 자원을 어떻게 사용하는지**를 이해하는 데 중점을 두었다.

특히 다음 흐름을 연결해서 이해할 수 있었다.

```text
모델 크기 확인
    ↓
필요한 VRAM 계산
    ↓
모델을 VRAM에 적재
    ↓
GPU에서 추론
    ↓
Token을 반복적으로 생성
    ↓
GPU / VRAM 상태 확인
```

또한 `nvidia-smi`에서 보이는 GPU 메모리와 PyTorch 내부의 `allocated`, `reserved`가 서로 다른 관점의 값이라는 것을 실습을 통해 확인했다.

이전에는 서버 자원을 생각할 때 주로 **CPU와 RAM**을 생각했지만, AI 모델을 운영할 때는 여기에 **GPU와 VRAM**이라는 중요한 자원이 추가된다는 것을 이해하게 되었다.

특히 모델의 크기가 커질수록 단순히 파일 용량만 증가하는 것이 아니라 실제 실행에 필요한 VRAM과 연산량도 크게 증가하기 때문에, AI 인프라에서는 **어떤 모델을 어떤 GPU에 올릴 수 있는지 판단하고 GPU 자원을 효율적으로 관리하는 능력**이 중요하다는 것을 배웠다.
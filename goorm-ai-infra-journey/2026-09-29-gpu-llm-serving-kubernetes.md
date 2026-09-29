# 🚀 GPU 기반 LLM 추론과 Kubernetes GPU 자원 관리

## 📌 오늘의 학습 주제

오늘은 LLM을 GPU에서 실행하기 위해 필요한 GPU의 기본 개념과 VRAM 사용량을 직접 확인하고,
모델의 파라미터 수와 정밀도에 따른 메모리 사용량을 계산해 보았다.

또한 CUDA OOM을 직접 발생시켜 GPU 메모리 관리 방식을 확인하고,
vLLM을 이용한 LLM Serving과 Kubernetes에서 GPU를 자원으로 관리하는 방법까지 학습했다.

전체적인 학습 흐름은 다음과 같다.

```text
GPU 상태 확인
    ↓
CPU / GPU 행렬 연산 비교
    ↓
GPU 비동기 연산 이해
    ↓
VRAM과 CUDA OOM
    ↓
모델 크기 계산
    ↓
FP16 / FP32 / INT4
    ↓
vLLM으로 LLM Serving
    ↓
Kubernetes GPU 자원 관리
```

---

# 1. GPU 상태 확인하기

GPU 상태는 `nvidia-smi` 명령어를 이용하여 확인할 수 있다.

```bash
nvidia-smi
```

이번 실습에서는 Tesla T4 GPU를 사용했다.

`nvidia-smi`에서는 다음과 같은 정보를 확인할 수 있다.

- GPU 모델
- VRAM 전체 용량
- 현재 VRAM 사용량
- GPU 사용률
- GPU 온도

CPU 환경과 비교하면 다음과 같이 생각할 수 있다.

| GPU | CPU / Linux |
|---|---|
| VRAM | RAM |
| VRAM 사용량 | `free -h`의 Memory 사용량 |
| GPU-Util | `top`의 CPU 사용률 |
| GPU 이름 | CPU 모델 |
| GPU 온도 | 별도의 sensor 정보 |

즉 CPU가 RAM을 사용하는 것처럼 GPU는 계산을 위해 VRAM을 사용한다.

---

# 2. CPU와 GPU 행렬곱 비교

LLM 내부에서는 매우 많은 행렬 연산이 수행된다.

간단하게 표현하면 다음과 같다.

```text
입력 Token
    ↓
숫자 Vector
    ↓
학습된 Weight와 행렬 연산
    ↓
Transformer Layer
    ↓
다음 Token 예측
```

GPU가 이러한 계산에 적합한 이유를 확인하기 위해
4096 × 4096 행렬곱을 CPU와 GPU에서 각각 실행해 보았다.

```python
import torch
import time

SIZE = 4096

# CPU
a_cpu = torch.randn(SIZE, SIZE)
b_cpu = torch.randn(SIZE, SIZE)

start = time.perf_counter()

c_cpu = torch.matmul(a_cpu, b_cpu)

cpu_time = time.perf_counter() - start


# GPU
a_gpu = a_cpu.to("cuda:0")
b_gpu = b_cpu.to("cuda:0")

torch.cuda.synchronize()

start = time.perf_counter()

c_gpu = torch.matmul(a_gpu, b_gpu)

torch.cuda.synchronize()

gpu_time = time.perf_counter() - start


print(f"CPU 시간 : {cpu_time:.6f} 초")
print(f"GPU 시간 : {gpu_time:.6f} 초")
print(f"GPU가 CPU보다 약 {cpu_time / gpu_time:.2f}배 빠름")
```

실습에서는 대략 다음과 같은 결과가 나왔다.

```text
CPU 시간 : 0.634992초
GPU 시간 : 0.165019초

GPU가 CPU보다 약 3.85배 빠름
```

GPU는 많은 계산을 병렬로 처리할 수 있기 때문에
대규모 행렬 연산이 반복되는 LLM에서 중요한 역할을 한다.

---

# 3. GPU 연산과 synchronize()

GPU 연산은 CPU 관점에서 비동기적으로 실행될 수 있다.

즉 CPU가 GPU에게 계산을 요청한 뒤
GPU 계산이 끝날 때까지 기다리지 않고 다음 코드를 실행할 수 있다.

```text
CPU
 │
 ├─ GPU에게 계산 요청
 │
 ├─ 다음 작업 진행
 │
 ▼

GPU
 │
 └─ 실제 계산 수행 중...
```

따라서 GPU의 실제 계산 시간을 측정할 때는 다음 명령어를 사용할 수 있다.

```python
torch.cuda.synchronize()
```

`synchronize()`를 제거하고 시간을 측정했을 때 실습에서는 다음과 같은 값이 나왔다.

```text
CPU : 0.569103초
GPU : 0.000235초

약 2425배 빠른 것처럼 측정
```

하지만 실제로 GPU가 2425배 빨라진 것은 아니다.

GPU 연산이 끝나기 전에 CPU가 시간 측정을 종료했기 때문에
실제 GPU 계산 시간보다 훨씬 짧게 측정된 것이다.

따라서 정확한 GPU 실행 시간을 측정하려면
GPU 작업이 끝날 때까지 기다리는 동기화 과정이 필요하다.

---

# 4. VRAM과 CUDA OOM

GPU에도 사용할 수 있는 메모리의 한계가 있다.

이를 직접 확인하기 위해 1GiB 크기의 Tensor를 반복적으로 GPU에 할당했다.

```python
import torch

blocks = []

for i in range(20):

    print(f"{i + 1} GiB 할당 시도...")

    x = torch.empty(
        268_435_456,
        dtype=torch.float32,
        device="cuda:0"
    )

    blocks.append(x)

    print(
        f"현재 allocated: "
        f"{torch.cuda.memory_allocated(0) / 1024**3:.2f} GiB"
    )
```

float32는 하나의 값이 4Byte이므로

```text
1 GiB
= 1,073,741,824 Byte

1,073,741,824 / 4
= 268,435,456
```

개의 float32 값을 생성하면 약 1GiB가 된다.

실습에서는 약 14GiB까지 할당된 후 추가 1GiB를 요청하면서 CUDA OOM이 발생했다.

오류 메시지에서는 다음 내용을 확인할 수 있었다.

```text
요청한 메모리 : 1024 MiB
GPU 전체 용량 : 14.56 GiB
남아 있던 메모리 : 470.81 MiB
```

즉,

```text
필요한 메모리
1024 MiB

>

남은 메모리
470.81 MiB

→ CUDA Out Of Memory
```

가 발생한 것이다.

---

# 5. memory_allocated / memory_reserved / nvidia-smi

PyTorch에서 GPU 메모리를 확인할 때 대표적으로 다음 값을 사용할 수 있다.

```python
torch.cuda.memory_allocated()
torch.cuda.memory_reserved()
```

### memory_allocated

현재 살아 있는 PyTorch Tensor가 실제로 사용하고 있는 GPU 메모리이다.

### memory_reserved

PyTorch가 앞으로 다시 사용할 수 있도록 확보해 둔 GPU 메모리까지 포함한다.

### nvidia-smi

GPU Driver 관점에서 프로세스가 사용하고 있는 GPU 메모리를 보여준다.

따라서 세 값은 서로 완전히 동일하지 않을 수 있다.

```text
PyTorch Tensor
     ↓
memory_allocated

PyTorch가 확보한 영역
     ↓
memory_reserved

GPU Driver에서 보는 전체 사용량
     ↓
nvidia-smi
```

---

# 6. empty_cache()

Tensor 변수를 삭제하더라도 PyTorch가 GPU 메모리를 캐시로 가지고 있을 수 있다.

```python
del blocks
del x
```

사용하지 않는 PyTorch GPU Cache를 반환할 때는 다음 명령어를 사용할 수 있다.

```python
torch.cuda.empty_cache()
```

중요한 점은 `empty_cache()`가 현재 사용 중인 Tensor를 삭제하는 명령어는 아니라는 것이다.

```text
Tensor가 사용 중인 메모리
→ empty_cache로 제거되지 않음

사용하지 않는 PyTorch Cache
→ empty_cache로 반환 가능
```

---

# 7. OOMKilled와 CUDA OOM 차이

Kubernetes에서 경험했던 `OOMKilled`와 GPU에서 발생하는 `CUDA OOM`은 서로 다른 문제이다.

| 구분 | OOMKilled | CUDA OOM |
|---|---|---|
| 부족한 자원 | 시스템 RAM / Container Memory | GPU VRAM |
| 감지 | Linux Kernel / cgroup | CUDA / PyTorch |
| 프로세스 | 강제로 종료될 수 있음 | GPU 메모리 할당 실패 예외 발생 |
| 흔적 | Pod 상태, `kubectl describe pod` 등 | Python traceback, CUDA OOM 오류 |
| 대표 메시지 | `OOMKilled` | `torch.OutOfMemoryError` |

간단하게 정리하면 다음과 같다.

```text
OOMKilled
→ RAM 부족
→ Process Kill

CUDA OOM
→ VRAM 부족
→ GPU Memory Allocation 실패
```

---

# 8. 모델 파라미터와 VRAM 계산

LLM이 GPU에 올라갈 때 필요한 Weight 메모리는 기본적으로 다음과 같이 계산할 수 있다.

```text
모델 크기
= Parameter 수 × Parameter 하나의 Byte 수
```

정밀도별 크기는 다음과 같다.

```text
FP32 = 4 Byte
FP16 = 2 Byte
INT4 = 4 bit = 0.5 Byte
```

예를 들어 7B 모델을 FP16으로 사용한다면

```text
7B × 2 Byte
= 약 14GB
```

의 Weight 메모리가 필요하다.

---

# 9. FP16과 FP32 실제 비교

Qwen2.5-0.5B-Instruct 모델을 FP16과 FP32로 각각 GPU에 올려
실제 GPU 메모리 사용량을 비교했다.

FP16 모델은 다음과 같이 로딩할 수 있다.

```python
from transformers import AutoModelForCausalLM
import torch

model_name = "Qwen/Qwen2.5-0.5B-Instruct"

model_fp16 = AutoModelForCausalLM.from_pretrained(
    model_name,
    dtype=torch.float16
).to("cuda:0")
```

모델의 실제 Parameter 수는 다음과 같이 계산할 수 있다.

```python
num_params = sum(
    p.numel()
    for p in model_fp16.parameters()
)

print(num_params)
```

FP16의 이론적인 Weight 크기는

```python
fp16_size = num_params * 2
```

로 계산할 수 있다.

같은 모델을 FP32로 실행하면

```python
model_fp32 = AutoModelForCausalLM.from_pretrained(
    model_name,
    dtype=torch.float32
).to("cuda:0")
```

Parameter 하나가 4Byte가 된다.

따라서

```text
FP16 = Parameter × 2 Byte

FP32 = Parameter × 4 Byte

FP32 / FP16
= 4 / 2
= 2배
```

가 되어 같은 모델이라도 FP32의 Weight 메모리 크기가 FP16의 약 2배가 된다.

---

# 10. 계산값과 실제 VRAM 사용량이 다른 이유

`Parameter 수 × Byte` 공식은 기본적으로 모델의 Weight 크기를 계산한다.

하지만 실제 GPU에서는 Weight 이외에도 추가 메모리가 사용될 수 있다.

```text
GPU VRAM

├─ Model Weight
├─ Buffer
├─ CUDA Context
├─ PyTorch 관련 메모리
└─ 기타 실행에 필요한 메모리
```

따라서 이론적으로 계산한 값과 실제 GPU 메모리 사용량에는 차이가 발생할 수 있다.

또한 `torch.cuda.memory_allocated()`와 `nvidia-smi`는 측정 범위가 서로 다르기 때문에
두 명령어에서 확인되는 값도 다를 수 있다.

---

# 11. 모델 크기별 GPU 사용 가능 여부

Parameter 수 × Byte 공식으로 모델의 Weight 크기를 계산해 보았다.

| Model | Precision | Weight 크기 | T4 15GB | A10G 24GB | A100 80GB |
|---|---|---:|:---:|:---:|:---:|
| 1.5B | FP16 | 3GB | O | O | O |
| 1.5B | INT4 | 0.75GB | O | O | O |
| 7B | FP16 | 14GB | O | O | O |
| 7B | INT4 | 3.5GB | O | O | O |
| 13B | FP16 | 26GB | X | X | O |
| 13B | INT4 | 6.5GB | O | O | O |

단, 위 표는 모델의 Weight만 단순 계산한 값이다.

실제 추론에서는 KV Cache, CUDA 관련 메모리 등 추가 VRAM이 필요하기 때문에
VRAM 용량과 모델 Weight 크기가 거의 같은 경우 실제 실행에서는 OOM이 발생할 수 있다.

---

# 12. GPU Compute Capability

GPU가 지원하는 연산 기능은 다음 명령어로 확인할 수 있다.

```python
import torch

print(torch.cuda.get_device_name(0))
print(torch.cuda.get_device_capability(0))
```

Tesla T4에서는 다음과 같이 확인할 수 있다.

```text
Tesla T4
(7, 5)
```

여기서 `(7, 5)`는 메모리가 7.5GB라는 의미가 아니다.

```text
T4

VRAM
→ GPU Memory 용량

Compute Capability 7.5
→ GPU가 지원하는 연산 기능의 세대
```

T4의 Compute Capability는 7.5이며,
T4에서는 BF16 대신 FP16을 사용하도록 vLLM 실행 시 다음 옵션을 사용할 수 있다.

```bash
--dtype half
```

여기서 `half`는 FP16을 의미한다.

---

# 13. vLLM으로 LLM을 서버로 실행

LLM을 Python Notebook 안에서 직접 호출하는 것뿐만 아니라
vLLM을 이용하면 모델을 API Server 형태로 실행할 수 있다.

```text
LLM Model
    ↓
vLLM
    ↓
LLM API Server
    ↓
HTTP Request
    ↓
Response
```

vLLM Server에서는 다음과 같은 API를 사용할 수 있다.

```text
/v1/models
/v1/chat/completions
```

예를 들어 사용자는 HTTP 요청을 보내 LLM과 통신할 수 있다.

```text
Client
   ↓ HTTP
vLLM Server
   ↓
GPU
   ↓
LLM
   ↓
Response
```

이렇게 모델을 Server 형태로 만들면 다른 Application에서도
주소를 통해 LLM을 호출할 수 있다.

---

# 14. Kubernetes에서 GPU 관리

Kubernetes는 CPU와 Memory뿐만 아니라 GPU도 자원으로 관리할 수 있다.

GPU Node에 NVIDIA Device Plugin 등을 구성하면 Kubernetes가 GPU를
다음과 같은 확장 자원으로 인식할 수 있다.

```text
nvidia.com/gpu
```

예를 들어 Node에 GPU가 2개 있다면 Kubernetes에서는 다음과 같이 관리할 수 있다.

```text
GPU Node

nvidia.com/gpu: 2
```

Pod에서 GPU가 필요하면 GPU 자원을 요청한다.

```yaml
resources:
  limits:
    nvidia.com/gpu: 1
```

그러면 Scheduler가 GPU를 사용할 수 있는 Node를 찾아 Pod를 배치한다.

---

# 15. GPU는 기본적으로 정수 단위로 요청

CPU는 여러 Pod가 세분화하여 나누어 사용할 수 있지만,
일반적인 Kubernetes GPU Extended Resource는 정수 단위로 요청한다.

예를 들어 GPU가 2개인 상황에서

```text
GPU 0 → Pod A
GPU 1 → Pod B

Pod C
  ↓
사용 가능한 GPU 없음
  ↓
Pending
```

처럼 될 수 있다.

GPU를 더 세분화하여 사용하는 방식으로는 MIG 등의 기술도 존재한다.

---

# 16. GPU Pod를 어느 Node에 배치할 것인가

실제 Kubernetes Cluster에는 GPU Node와 일반 CPU Node가 함께 존재할 수 있다.

```text
Kubernetes Cluster

├─ CPU Node
├─ CPU Node
└─ GPU Node
      └─ Tesla T4
```

GPU가 필요한 Pod는 GPU Node로 보내야 한다.

이를 위해 Node에 Label을 지정하고 `nodeSelector`를 사용할 수 있다.

```yaml
nodeSelector:
  accelerator: tesla-t4
```

전체적인 흐름은 다음과 같다.

```text
GPU Node
   ↓
Label 설정

accelerator=tesla-t4

   ↓

Pod
   ↓
nodeSelector

   ↓

Kubernetes Scheduler

   ↓

GPU Node에 Pod 배치
```

또한 `taint`와 `toleration`을 이용하면
특정 Node에 허용된 Pod만 배치되도록 제한할 수도 있다.

---

# 17. 오늘 배운 내용 전체 흐름

오늘 학습한 내용을 하나의 흐름으로 연결하면 다음과 같다.

```text
LLM
 │
 │ 수많은 행렬 연산
 ▼
GPU 필요
 │
 ▼
Model Weight를 VRAM에 저장
 │
 ├─ FP32
 ├─ FP16
 └─ INT4
 │
 ▼
모델 크기에 따라 VRAM 사용량 결정
 │
 ▼
VRAM 부족
 │
 └─ CUDA OOM
 │
 ▼
vLLM으로 Model Serving
 │
 ▼
API Server
 │
 ▼
Kubernetes에 배포
 │
 ▼
nvidia.com/gpu 요청
 │
 ▼
Scheduler
 │
 ▼
GPU Node에 Pod 배치
```

---

# ✍️ 오늘 배운 점

오늘 수업에서는 단순히 GPU에서 LLM을 실행하는 것뿐만 아니라,
**LLM이 GPU 자원을 어떻게 사용하는지부터 Kubernetes에서 GPU를 어떻게 관리하는지까지**
전체적인 흐름을 학습했다.

특히 모델의 Parameter 수와 FP16, FP32, INT4 같은 정밀도를 이용하여
모델 Weight가 어느 정도의 VRAM을 필요로 하는지 직접 계산해 보았다.

또한 실제 Tensor를 반복적으로 생성하여 CUDA OOM을 발생시키면서
GPU VRAM의 한계와 PyTorch의 `memory_allocated`, `memory_reserved`,
`empty_cache()`의 차이도 확인할 수 있었다.

CPU와 GPU의 행렬곱 성능을 직접 비교하면서
LLM에서 GPU가 사용되는 이유를 이해할 수 있었고,
GPU 연산이 비동기적으로 동작하기 때문에 정확한 성능 측정에는
`synchronize()`가 필요하다는 것도 알게 되었다.

마지막으로 vLLM을 통해 LLM을 API Server로 만들고,
Kubernetes에서는 NVIDIA Device Plugin을 통해 GPU를 `nvidia.com/gpu`라는
자원으로 인식하여 필요한 Pod에 할당한다는 것을 배웠다.

이번 학습을 통해 다음의 연결 관계를 이해할 수 있었다.

> **모델의 크기와 정밀도 → 필요한 VRAM → GPU에서 LLM 실행 → vLLM Serving → Kubernetes GPU Scheduling**

앞으로 LLM 서비스를 운영할 때는 단순히 모델을 실행하는 것뿐만 아니라
**모델 크기, GPU VRAM, 추론 방식, GPU 자원 할당 및 Kubernetes Scheduling까지 함께 고려해야 한다.**
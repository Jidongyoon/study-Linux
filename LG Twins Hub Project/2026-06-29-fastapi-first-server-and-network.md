# 2026-06-29 FastAPI 첫 서버 구축 및 네트워크 연결

## 목표

LG Twins Hub 프로젝트의 첫 번째 FastAPI 서버를 구축하고,
Windows 브라우저에서 Rocky Linux VM의 FastAPI 서버에 접속하는 과정을 실습하였다.

---

# 프로젝트 구조

```text
lg-twins-hub/
├── app/
│   └── main.py
├── config/
├── docs/
├── logs/
├── nginx/
├── scripts/
├── static/
├── templates/
├── tests/
├── requirements.txt
└── venv/
```

---

# Python 가상환경

가상환경 활성화

```bash
source venv/bin/activate
```

설치된 패키지 확인

```bash
pip list
```

requirements 생성

```bash
pip freeze > requirements.txt
```

---

# FastAPI 기본 코드

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello LG Twins!"}
```

---

# Uvicorn 실행

처음 실행

```bash
uvicorn app.main:app --reload
```

외부에서 접속 가능하도록 수정

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

옵션 의미

- --host : 접속 허용 주소
- --port : 사용할 포트
- --reload : 코드 변경 시 자동 재시작

---

# localhost 개념 이해

처음에는

```
http://localhost:8000
```

으로 접속하였다.

하지만 localhost는 현재 사용하는 컴퓨터 자신을 의미한다.

즉,

Windows에서

```
localhost
```

는

```
Windows 자신의 127.0.0.1
```

이다.

FastAPI는 Rocky Linux VM에서 실행되고 있으므로

Windows에서는

```
http://192.168.111.100:8000
```

으로 접속해야 한다.

---

# VM IP 확인

```bash
ip addr
```

또는

```bash
hostname -I
```

결과

```
192.168.111.100
```

---

# FastAPI 정상 동작 확인

VM 내부에서

```bash
curl http://127.0.0.1:8000
```

결과

```json
{"message":"Hello LG Twins!"}
```

애플리케이션이 정상 동작함을 확인하였다.

---

# 서버 포트 확인

```bash
ss -tulnp | grep 8000
```

실행 중일 때

```
LISTEN 0.0.0.0:8000
```

이 표시된다.

처음에는 아무 결과가 나오지 않았는데,
원인은 Uvicorn 서버를 종료한 상태였기 때문이다.

웹 서버는 실행 상태를 계속 유지해야 한다.

---

# 방화벽 확인

```bash
sudo firewall-cmd --list-all
```

포트 확인

```bash
sudo firewall-cmd --list-ports
```

FastAPI 실습을 위해

```bash
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload
```

명령으로 8000 포트를 허용하였다.

---

# VMware 네트워크 확인

Virtual Machine Settings

↓

Network Adapter

↓

NAT 방식 사용

VM IP를 이용하여 Windows 브라우저에서 접속하였다.

---

# Swagger UI 확인

FastAPI는 자동으로 API 문서를 생성한다.

접속

```
http://192.168.111.100:8000/docs
```

Swagger UI 화면이 정상적으로 출력되는 것을 확인하였다.

---

# 오늘 사용한 주요 명령어

```bash
source venv/bin/activate

pip list

pip freeze > requirements.txt

uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

curl http://127.0.0.1:8000

ip addr

hostname -I

ss -tulnp | grep 8000

sudo firewall-cmd --list-all

sudo firewall-cmd --list-ports

sudo firewall-cmd --permanent --add-port=8000/tcp

sudo firewall-cmd --reload
```

---

# 오늘 배운 핵심

- Python Virtual Environment(venv)
- FastAPI 프로젝트 생성
- Uvicorn 실행 방법
- localhost의 의미
- Windows와 VM의 네트워크 차이
- VM IP를 이용한 접속 방법
- FastAPI JSON 응답
- Swagger UI 사용
- 웹 서버는 종료되지 않고 계속 실행되어야 함
- curl, ip addr, ss를 이용한 기본적인 트러블슈팅 방법

---

# 느낀 점

FastAPI 실행 자체보다
애플리케이션과 네트워크를 구분하여 문제를 해결하는 과정이 가장 인상 깊었다.

처음에는 FastAPI 문제라고 생각했지만,
실제로는 localhost의 의미, VM IP, 방화벽, 서버 실행 여부 등을 하나씩 확인하면서 원인을 찾을 수 있었다.

앞으로 Nginx, Docker, Kubernetes를 학습할 때도
오늘 익힌 네트워크와 트러블슈팅 방식이 기본이 될 것 같다.

---

# 다음 목표

- 여러 개의 API(Route) 생성
- Path Parameter 학습
- Query Parameter 학습
- Pydantic Model 적용
- CRUD API 구현
- Nginx Reverse Proxy 적용
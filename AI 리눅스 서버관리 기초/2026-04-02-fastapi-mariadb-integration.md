# 🧦 Full-Stack E-Commerce Data Pipeline Project
> **FastAPI + SQLAlchemy + MariaDB를 활용한 주문 자동화 시스템 구축**

본 프로젝트는 단순한 코드 작성을 넘어, 클라이언트-서버-데이터베이스로 이어지는 **데이터의 흐름(Data Flow)**과 **관계형 데이터베이스(RDBMS)**의 무결성을 실습하기 위해 구축되었습니다.

---

## 🏗 1. System Architecture (시스템 구조)

데이터는 아래와 같은 경로로 흐르며, 각 단계는 독립적인 역할을 수행합니다.

1.  **Client (Python Script)**: `requests` 라이브러리를 통해 주문/사용자 데이터를 생성하여 서버로 쏩니다.
2.  **Server (FastAPI)**: 들어온 데이터를 검증하고, SQLAlchemy(ORM)를 통해 DB 명령어로 변환합니다.
3.  **Database (MariaDB)**: 최종적으로 데이터를 저장하며, 테이블 간의 관계(Foreign Key)를 감시합니다.



---

## 💡 2. Key Concepts & Lessons (핵심 개념 정리)

### ① API Mapping (연결의 원리)
클라이언트의 `POST` 요청이 서버의 특정 함수와 매핑되는 원리를 이해했습니다.
* **Endpoint**: `/orders`라는 주소가 서버와 클라이언트 간의 약속된 통로입니다.
* **Payload**: JSON 형식의 데이터 묶음으로, 서버가 기다리는 '설계도(Schema)'와 일치해야 합니다.

### ② Database Integrity (데이터 무결성)
* **Foreign Key (외래 키)**: `orders` 테이블의 유저는 반드시 `users` 테이블에 존재해야 한다는 규칙입니다. 
* **TRUNCATE**: 테이블의 구조는 남기고 데이터만 빛의 속도로 비우며, ID(Auto Increment)를 초기화하는 명령어입니다. `SET FOREIGN_KEY_CHECKS = 0`을 통해 일시적으로 제약을 끄고 작업하는 법을 익혔습니다.

### ③ Automation with Python (`def` & `sys.argv`)
* **`def`**: 반복되는 로직(주문 생성)을 하나의 함수로 정의하여 코드의 재사용성을 높였습니다.
* **`sys.argv`**: 실행 시점에 숫자를 입력받아(예: `python3 script.py 100`) 작업량을 유연하게 조절하는 법을 배웠습니다.

---

## 🛠 3. Troubleshooting (트러블슈팅 기록)

| 현상 | 원인 | 해결책 |
| :--- | :--- | :--- |
| **500 Internal Error** | DB 테이블 미생성 상태에서 접근 시도 | `create_all()` 호출 위치를 모델 정의 하단으로 이동 |
| **Empty Set (Empty Table)** | 스크립트 실행 시 인자(Argument) 누락 | `python3 create_orders.py 100`으로 명확한 수치 하달 |
| **Duplicate Data** | 스크립트 중복 실행으로 데이터 누적 | `TRUNCATE`를 통한 데이터 초기화 후 단일 시딩 수행 |

---

## 🚀 4. Execution Guide (실행 순서)

나중에 다시 실습할 때 이 순서대로 명령어를 입력하세요.

1. **서버 가동**: `uvicorn main:app --reload`
2. **기본 데이터 입고**: `python3 init_users_socks.py` (고객 1,000명 / 양말 500개)
3. **주문 폭격 (100건)**: `python3 create_orders.py 100`
4. **장부 확인**: `python3 show_orders.py`
5. **DB 직접 검증**: `SELECT * FROM orders;` (MariaDB 접속 상태)

---

## 📝 Master's Final Review
> "코드는 단순한 글자가 아니라 시스템을 움직이는 명령이다. 서버가 500 에러를 뱉을 때 그것은 에러가 아니라 '나 여기 고장 났어'라고 보내는 SOS 신호임을 이해하게 되었다. 이제 나는 데이터의 생성부터 조회까지 전체 사이클을 제어할 수 있다."
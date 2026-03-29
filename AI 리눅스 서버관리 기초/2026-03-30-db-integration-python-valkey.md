# 🚀 Cloud Infrastructure: Database & Application Integration
MariaDB(RDBMS)와 Valkey(In-Memory DB), 그리고 Python 애플리케이션 연동 실습 기록

## 1. MariaDB: Relational Data Management
데이터의 무결성을 유지하며 복잡한 관계를 처리하는 관계형 데이터베이스 실습.

### ✅ Key Accomplishments
- **Multi-Table JOIN**: `users`, `socks`, `orders` 테이블을 결합하여 가독성 있는 주문 리포트 생성.
- **Data Aggregation**: `GROUP BY`와 `SUM`, `COUNT`를 활용해 사용자별 총 구매 금액 및 베스트셀러 상품 통계 산출.
- **Data Filtering**: `WHERE` 절과 `LEFT JOIN`을 활용해 고가 상품 주문 내역 및 주문 이력이 없는 휴면 고객(Null Data) 식별.

> **Insight**: 모든 집계의 기준은 동명이인 방지를 위해 고유 식별자(ID)를 포함해야 함을 체득함.

---

## 2. Python-DB Connectivity (pymysql)
애플리케이션 레벨에서 데이터베이스를 제어하는 백엔드 인프라의 기초 확립.

### ✅ Debugging Experience
- **Environment Management**: 리눅스 쉘과 파이썬 인터프리터의 실행 환경 차이 이해.
- **Error Handling**: `NameError` 트러블슈팅을 통해 모듈 임포트(`import pymysql`) 및 스크립트 실행 순서의 중요성 확인.
- **Automation**: SQL 쿼리를 파이썬 스크립트화하여 데이터 조회 프로세스 자동화 구현.

---

## 3. Valkey (In-Memory DB): High Performance Caching
실시간 데이터 처리와 성능 최적화를 위한 인메모리 데이터베이스 실습.

### ✅ Core Operations
- **Key-Value Store**: 비정형 데이터를 초고속으로 저장하고 읽는 `SET/GET` 메커니즘 이해.
- **Atomic Operation**: `INCR` 명령어를 활용하여 동시성 이슈 없이 실시간 카운터를 처리하는 방식 학습.

### 🎯 Architecture Strategy
- **Persistence (Disk)**: 영구 보관이 필요한 정형 데이터는 **MariaDB**에 저장.
- **Performance (RAM)**: 빠른 응답 속도가 필요한 캐시 데이터는 **Valkey**에 배치하여 시스템 전체 성능 최적화.

---

## ⚾ Future Project: LG Twins Data Platform
- **Goal**: 오늘 학습한 DB 기술을 활용해 LG 트윈스 선수 기록 및 실시간 경기 통계 서비스 구축 예정.
- **Tech Stack**: MariaDB(선수 기록) + Valkey(실시간 스코어/좋아요) + Python(Backend).


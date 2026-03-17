# 🌐 HTTP Core Fundamentals: 핵심 요약 및 정리

HTTP(HyperText Transfer Protocol)의 주요 특성과 구조, 그리고 실무에서 자주 사용되는 메서드 및 상태 코드를 표 형식으로 정리함.

## 🚀 1. HTTP의 4대 핵심 특성
| 특성 | 설명 | 비고 |
| :--- | :--- | :--- |
| **Connectionless** | 클라이언트 요청에 서버가 응답을 마치면 즉시 연결을 끊음 | 자원 낭비를 줄이지만 재연결 오버헤드 존재 |
| **Stateless** | 서버가 클라이언트의 이전 상태를 기억하지 않음 | 서버 확장성이 좋으나 쿠키/세션으로 보완 필요 |
| **Media Independent** | 어떤 데이터 타입(HTML, 이미지, JSON 등)도 전송 가능 | `Content-Type` 헤더로 데이터 종류 명시 |
| **Client-Server** | 클라이언트가 요청(Request)하고 서버가 응답(Response)함 | 명확한 역할 분담 및 독립적 발전 가능 |

## 📑 2. HTTP 메시지 구조 (Message Structure)
| 구성 요소 | 설명 | 예시 |
| :--- | :--- | :--- |
| **Start Line** | 요청(Method, Path) 또는 응답(Status) 정보 | `GET /login HTTP/1.1` 또는 `HTTP/1.1 200 OK` |
| **Headers** | 메시지에 대한 추가 정보 (메타데이터) | `Host: localhost`, `Content-Type: text/html` |
| **Empty Line** | 헤더와 본문을 구분하는 빈 줄 | (헤더 끝을 알리는 필수 구분선) |
| **Body** | 실제 전송할 데이터 내용 (본문) | HTML 코드, JSON 데이터, 이미지 파일 등 |



[Image of HTTP request and response message structure]


## 🛠️ 3. 주요 HTTP 메서드 (Methods)
| 메서드 | 역할 | 비고 |
| :--- | :--- | :--- |
| **GET** | 리소스 조회 (서버 데이터 요청) | 데이터가 URL에 노출됨 (Query String) |
| **POST** | 리소스 생성 (데이터 등록) | 데이터가 본문(Body)에 담겨 전송됨 |
| **PUT** | 리소스 전체 수정 (덮어쓰기) | 해당 경로의 자원을 새 데이터로 완전히 교체 |
| **PATCH** | 리소스 부분 수정 | 데이터의 일부분만 변경할 때 사용 |
| **DELETE** | 리소스 삭제 | 특정 자원의 삭제 요청 |

## 🚨 4. 주요 HTTP 상태 코드 (Status Codes)
| 코드 대역 | 의미 | 대표 사례 |
| :--- | :--- | :--- |
| **2xx (Success)** | 요청 성공 | **200 OK** (성공), **201 Created** (생성 완료) |
| **3xx (Redirection)** | 추가 동작 필요 | **301 Moved** (주소 이전), **304 Not Modified** (캐시 사용) |
| **4xx (Client Error)** | 클라이언트 측 오류 | **404 Not Found** (경로 없음), **403 Forbidden** (권한 없음) |
| **5xx (Server Error)** | 서버 측 오류 | **500 Internal Error** (서버 로직 에러), **503 Service Unavailable** |

---
> **Master's Log**: 
> "HTTP의 단순하면서도 강력한 구조가 오늘날 웹의 기반이 되었음을 이해함. 특히 Stateless와 Connectionless 특성은 인프라 설계 시 부하 분산과 세션 관리를 고민하게 만드는 핵심 원리이며, 상태 코드는 서버와 클라이언트 간의 가장 명확한 대화 수단이다."




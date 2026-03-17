# 📚 Infrastructure & Network Practice (2026-03-17)

## 🏗️ 1. Infrastructure Architecture
| 구분 | 구성 요소 | 주요 역할 |
| :--- | :--- | :--- |
| **Web Server** | **Nginx** | HTTP 통신 전문가, 리버스 프록시, 정적 파일 고속 전송 |
| **WAS** | **Django (Gunicorn)** | 비즈니스 로직 처리, 파이썬 기반 어플리케이션 구동 |
| **Interface** | **lo / eth0** | 내부 전용 통로(lo)와 외부 대문(eth0)의 분리 운영 |

## ⚠️ 2. Essential Troubleshooting (Must Memorize)
| 에러 메시지 | 발생 원인 (OS 레벨) | 해결 방법 |
| :--- | :--- | :--- |
| **Address already in use** | 80/8000 포트가 이미 메모리상 소켓에 점유됨 | `sudo fuser -k [포트]/tcp`로 프로세스 종료 |
| **could not bind** | 커널에 `bind()` 시스템콜 요청 시 중복 할당 거부 | 기존 서비스 중지(`stop`) 후 재시작 |
| **Permission Denied** | 관리자 권한 없이 시스템 포트(0-1023) 접근 | 명령어 앞에 `sudo` 반드시 추가 |
| **Window Resize Error** | `iptraf-ng` 실행 터미널 규격 미달 | 터미널 창 가로 80, 세로 24 이상 확대 |

## 🌐 3. Network Deep Dive
| 핵심 개념 | 상세 설명 | 비고 |
| :--- | :--- | :--- |
| **Memory Socket** | 커널 메모리에 기록된 IP/Port 통신 장부 | 프로세스 종료 시 메모리에서 소멸 |
| **Loopback (lo)** | `127.0.0.1` (우리 집 인터폰) | 서버 내부 프로세스 간 보안 통신 |
| **Ethernet (eth0)** | 서버 실제 IP (우리 집 대문) | 외부 사용자 접속 및 패킷 송수신 |
| **curl -v** | HTTP 요청/응답 전체 과정 시각화 | 빈 응답(Empty Response) 디버깅 필수 도구 |

## 🧪 4. Practice Log & Analysis
| 테스트 상황 | 결과 및 현상 | 분석 결과 |
| :--- | :--- | :--- |
| **Root URL 요청** | `curl http://localhost:8000` -> 출력 없음 | `/` 경로에 매핑된 데이터가 없는 상태 (정상) |
| **Login URL 요청** | `curl .../login/` -> HTML 소스 출력 | 서버가 로그인 폼(HTML 설계도)을 전송함 |
| **Traffic Monitor** | `iptraf-ng` 패킷 카운트 급상승 | `ab` 테스트에 따른 실시간 네트워크 부하 관찰 |

---
> **Master's Log**: 웹 서버는 최전방에서 HTTP 프로토콜을 처리하고, 내부 로직은 루프백 인터페이스를 통해 안전하게 소통하는 인프라 구조를 이해함. 특히 `bind()` 시스템콜과 메모리상 소켓의 관계를 통해 포트 충돌 원리를 파악함.

# 🚀 Samsung Electronics Stock Monitoring AI Agent

> **지능형 AI 에이전트의 구조를 이해하고 구현하는 주가 모니터링 자동화 프로젝트**
> 본 프로젝트는 단순한 규칙 기반 코딩을 넘어, 데이터 수집부터 판단, 실행까지 스스로 처리하는 독립적인 **AI 에이전트(AI Agent)**의 기본 뼈대를 학습하고 구축한 결과물입니다. ---

## 📌 1. AI 에이전트 핵심 개념

### 💡 왜 AI 에이전트인가? | 구분 | Before AI (규칙 중심) | After AI (지능 중심) |
| :--- | :--- | :--- |
| **실행 조건** | 정해진 시간, 명확한 수치 | 데이터 맥락에 따른 유연한 판단 |
| **업무 수행** | 단순 계산, 고정된 공식 | 데이터 분석, 요약, 새로운 가공 |
| **업무 완료** | 단순 저장 및 일괄 발송 | 대상 판단 및 최적 경로 선택 |

* **대화형 AI (Chat LLM):** 사용자가 질문을 던져야만 답하는 수동적인 구조
* **행동형 에이전트 (AI Agent):** 스스로 시계를 보며 판단하고 외부 환경(API, 메신저)과 상호작용하는 주체적인 구조 ---

## 🛠️ 2. 에이전트 제작 프로세스 & 아키텍처 에이전트는 **[기획 ➡️ 프롬프트 ➡️ 개발 ➡️ 실행]**의 4단계 프로세스를 거쳐 완성되었습니다. ```
 [1. 데이터 수집: yfinance] 
         │ (5분마다 주가 데이터 자동 조회)
         ▼
 [2. 조건 판단: 1주일 평균가 대비 ±2% 변동성 검사]
         │ (특이사항 발생 시 트리거 발동)
         ▼
 [3. 자율 행동: 웹후크(Webhook) 알림 전송]
```

1. **수집(Collect):** `yfinance`를 사용하여 삼성전자(`005930.KS`)의 최근 7일간 종가 평균값 및 현재가를 수집합니다.
2. **판단(Judge):** 현재 주가가 1주일 평균가보다 2% 이상 변동(상승/하락)했는지 검증합니다.
3. **행동(Action):** 트리거 조건 충족 시, 포맷팅된 메시지를 생성하여 외부 메신저로 실시간 발송합니다.

---

## 💻 3. 실행 환경 및 설치 방법

### 📦 개발 환경 * **Language:** Python 3.14.5 * **OS:** Windows 11 ### 🛠️ 필수 라이브러리 설치
터미널 창을 열고 아래 명령어를 입력하여 필요한 패키지를 설치합니다.
```bash
pip install yfinance schedule requests
```

### 🚀 에이전트 실행 명령어
```bash
# 1. 에이전트가 저장된 폴더 경로로 이동
cd C:\Users\dongp\.gemini\antigravity\scratch\samsung_stock_monitor

# 2. 파이썬 파일 시동
python monitor.py
```

---

## 🚨 4. 트러블슈팅 (Troubleshooting)

### 📌 구글 챗 웹후크 권한 제한 이슈 * **문제 상황:** 구글 챗 스페이스에서 웹후크 생성 시 `"웹후크 관리는 제한되어 있습니다"` 메시지와 함께 생성 버튼이 비활성화됨. (구글 개인 계정 제한 및 조직 보안 정책 원인) * **해결 방법:** 외부 연동 제한이 없고 개발자 친화적인 **디스코드(Discord) 웹후크 API**로 우회하여 알림 시스템 정상화 성공.
* **코드 수정:** 1. `GOOGLE_CHAT_WEBHOOK_URL` 변수에 디스코드 웹후크 주소 대입.
  2. 디스코드 메시지 전송 규격에 맞춰 JSON 파라미터 키값을 `text`에서 `content`로 변경 완료.

---

## 📂 5. 전체 소스 코드 (`monitor.py`)

```python
import yfinance as yf
import requests
import schedule
import time

# 디스코드 웹후크 URL (구글 챗 제한으로 인해 디스코드 우회)
DISCORD_WEBHOOK_URL = "YOUR_DISCORD_WEBHOOK_URL_HERE"

def get_weekly_average():
    """최근 7일간의 종가 평균을 구합니다."""
    ticker = yf.Ticker("005930.KS")
    df = ticker.history(period="7d")
    return df['Close'].mean()

def check_stock_and_alert():
    """주가를 모니터링하고 변동률에 따라 알림을 발송합니다."""
    try:
        weekly_avg = get_weekly_average()
        ticker = yf.Ticker("005930.KS")
        current_price = ticker.history(period="1d")['Close'].iloc[-1]
        
        # 변동률 계산
        change_percent = ((current_price - weekly_avg) / weekly_avg) * 100
        
        # 임계값 판단 (±2% 이상 변동 시)
        if abs(change_percent) >= 2:
            message = (
                f"🚨 **삼성전자 주가 변동 알림**\n"
                f"- 현재가: {current_price:,}원\n"
                f"- 1주 평균가: {weekly_avg:,.0f}원\n"
                f"- 변동률: {change_percent:.2f}%"
            )
            
            # 디스코드 API 규격(content)에 맞춰 데이터 전송
            response = requests.post(DISCORD_WEBHOOK_URL, json={"content": message})
            
            if response.status_code == 204:
                print(f"[{time.strftime('%Y-%m-%d %H:%M:%S')}] 디스코드 알림 전송 완료!")
            else:
                print(f"알림 전송 실패 (Status Code: {response.status_code})")
                
    except Exception as e:
        print(f"에러 발생: {e}")

# 5분마다 주가 감시 태스크 실행
schedule.every(5).minutes.do(check_stock_and_alert)

print("🚀 삼성전자 모니터링 에이전트 작동 시작... (5분마다 체크)")

# 백그라운드 무한 루프 실행
while True:
    schedule.run_pending()
    time.sleep(1)
```
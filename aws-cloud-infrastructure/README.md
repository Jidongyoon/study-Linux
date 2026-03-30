# ☁️ AWS Cloud Journey: Zero to SAA
> **"AWS 프리티어를 활용한 클라우드 인프라 구축 및 데이터 플랫폼 마이닝 프로젝트"**

본 저장소는 AWS(Amazon Web Services)의 다양한 서비스를 실습하고, 이를 활용하여 실제 데이터(LG 트윈스 통계 등)를 수집·관리하는 인프라를 구축하는 과정을 기록합니다. 단순 이론 공부를 넘어 **SAA(Solutions Architect Associate)** 자격 취득과 실무 역량 강화를 목표로 합니다.

---

## 🛠️ Tech Stack
- **Cloud**: AWS (EC2, VPC, EBS, IAM, RDS 등)
- **OS**: Ubuntu 24.04 LTS
- **Database**: MariaDB
- **Language**: Python (Web Crawling & Data Processing)
- **Monitoring**: htop, AWS Budgets, CloudWatch

---

## 🚀 Today's Achievement (2026-03-30)
### Phase 1: Foundations & Security
- **Root Security**: MFA(Multi-Factor Authentication) 및 패스키 등록을 통한 보안 강화.
- **Cost Control**: AWS Budgets 설정을 통한 실시간 결제 알람 구축 (프리티어 초과 방지).
- **Provisioning**: 시드니 리전(ap-southeast-2) 내 **t3.micro** 인스턴스 생성 및 가동.
- **System Audit**: `free`, `df`, `htop`, `netstat` 명령어를 활용한 인프라 리소스 및 네트워크 포트 정밀 실사 완료.

---

## 📈 Roadmap
- [x] AWS 계정 보안 및 비용 통제 설정
- [x] EC2 인스턴스(Ubuntu) 프로비저닝 및 터미널 접속
- [ ] **(Upcoming)** MariaDB 설치 및 보안 강화 (Binding Address 설정)
- [ ] **(Upcoming)** Python 크롤링 데이터 파이프라인 구축
- [ ] **(Upcoming)** SAA-C03 이론 지식 서비스 매핑 및 아카이빙

---

## ✍️ Master's Log
"클라우드라는 생소한 세계에 첫발을 내디뎠다. 127.0.0.1(Local)과 0.0.0.0(Public)의 차이를 이해하는 것부터가 보안의 시작임을 배웠다. 시드니에 띄운 나의 첫 서버가 LG 트윈스의 승리 데이터를 담는 거대한 데이터 센터로 변모하는 과정을 기록해 나갈 것이다."
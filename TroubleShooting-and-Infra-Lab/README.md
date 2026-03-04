# 🛠️ Server Break & Fix Log
> "고장 내봐야 배운다." - 리눅스 서버 파괴 및 복구 실습 기록 보관소입니다.

이 레포지토리는 Rocky Linux 환경에서 발생하는 다양한 장애 상황을 직접 재현하고 해결한 과정을 기록합니다.

---

## 🚀 실습 환경
- **OS**: Rocky Linux 9
- **Server**: Apache HTTPD
- **Host OS**: Windows 11 (Terminal access via SSH)

---

## 🛡️ 트러블슈팅 사례 (Case Studies)

### 📂 Case 01. 방화벽과 성문 (Network Port)
- **발생 날짜**: 2026-03-04
- **증상**: 서비스는 Active인데 외부 브라우저에서 접속 안 됨.
- **원인**: `firewalld`에서 80번 포트 미개방.
- **해결**: 
  - `firewall-cmd --permanent --add-service=http`
  - `firewall-cmd --reload` 후 접속 확인.

### 💾 Case 02. 디스크의 비명 (Disk Full)
- **발생 날짜**: 2026-03-04
- **증상**: 시스템 용량 부족으로 인한 쓰기 제한.
- **원인**: `/tmp` 디렉토리에 10GB 대형 파일 생성 테스트.
- **해결**:
  - `df -h`로 전체 용량 확인, `du -sh`로 범인 폴더 검거.
  - `rm -rf /tmp/big_boss_file`로 용량 복구.

### ⚙️ Case 03. 잘못된 설계도 (Config Syntax Error)
- **발생 날짜**: 2026-03-04
- **증상**: 아파치 재시작 실패 (`Job for httpd.service failed`).
- **원인**: `httpd.conf` 내 `DocumentRoot` 경로 오타.
- **해결**:
  - `httpd -t` 명령어로 오타 줄(124번 라인) 확인.
  - `vi` 편집기로 경로 원복 후 `Syntax OK` 확인 및 재시작.

### 🔒 Case 04. [여기에 제목을 입력하세요]
- **발생 날짜**: 
- **증상**: 
- **원인**: 
- **해결**: 

---

## 🛠️ 핵심 도구 모음 (Toolbox)
| 명령어 | 용도 |
|:---:|:---|
| `systemctl` | 서비스 상태 관리 |
| `journalctl -xe` | 시스템 로그 상세 확인 |
| `tail -f` | 실시간 로그 모니터링 |
| `df / du` | 디스크 용량 분석 |
| `netstat / ss` | 네트워크 포트 점유 확인 |

---


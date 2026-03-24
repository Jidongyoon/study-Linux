# 🐧 Linux Package Management & Troubleshooting Master Note

> **Date**: 2026-03-24
> **Topic**: Package Architecture (Ubuntu vs RedHat) & Error Diagnosis

---

## 1. 🔍 엔지니어의 시스템 진단 파이프라인
시스템에 문제가 생겼을 때, 아래 **4단계 레이어**를 순차적으로 점검하여 원인을 파악합니다.

### 📂 패키지 설치 실패 원인 4대 분류
| 레이어 | 주요 원인 | 진단 명령어 |
| :--- | :--- | :--- |
| **1. Network** | 인터넷 단절, 방화벽 차단 | `ping 8.8.8.8`, `curl -I [URL]` |
| **2. Repository** | 저장소 URL 오타, 메타데이터 불일치 | `cat /etc/apt/sources.list`, `yum repolist` |
| **3. Resource** | 디스크 용량 100%, 권한(sudo) 없음 | `df -h`, `du -sh /var/lib/apt/lists` |
| **4. Dependency** | 패키지 버전 충돌, 의존성 족보 꼬임 | `apt show [pkg]`, `yum deplist [pkg]` |

---

## 2. 🏗️ Ubuntu (APT 계열) 구조 파헤치기
우분투는 **'전단지(목록)'**와 **'실제 물건(패키지)'** 관리가 명확히 분리되어 있습니다.

### 📍 주요 설정 경로 및 역할
1. **메인 주소록**: `/etc/apt/sources.list`
   - 시스템 기본 패키지 정보가 담긴 메인 설정 파일.
2. **외부 주소록**: `/etc/apt/sources.list.d/*.list`
   - Docker, Chrome 등 외부 소프트웨어 전용 설정 보관.
3. **전단지 저장소 (Metadata)**: `/var/lib/apt/lists/`
   - `apt update` 실행 시 원격 서버에서 받아온 패키지 정보가 저장되는 곳.
4. **임시 창고 (Cache)**: `/var/cache/apt/archives/`
   - 설치 전 `.deb` 파일이 잠시 머무는 곳. `apt clean`으로 비울 수 있음.

### 🔄 `apt` 설치 내부 메커니즘
- **Step 1**: `sources.list`에서 저장소 URL 확인.
- **Step 2**: `apt update`로 최신 패키지 목록(전단지)을 로컬로 동기화.
- **Step 3**: `apt search`로 로컬에 저장된 전단지 내에서 키워드 검색.
- **Step 4**: `.deb` 파일을 다운로드 후, 로우레벨 도구인 `dpkg`를 통해 실제 설치.

---

## 3. 🎩 RedHat (YUM 계열) 특징 및 차이점
레드햇 계열(CentOS, Rocky 등)은 우분투보다 패키지 명칭과 관리가 엄격합니다.

* **설정 경로**: `/etc/yum.repos.d/*.repo` (각 저장소를 개별 파일로 관리)
* **파일 형식**: `INI` 스타일 (섹션 `[]`, `baseurl`, `enabled` 옵션 사용)
* **패키지 명칭**: 매우 구체적인 정식 명칭 사용 필수
  - ❌ `jdk` (이름이 모호하여 에러 발생 가능)
  - ✅ `java-1.8.0-openjdk.x86_64` (정확한 명칭)

---

## 🛠️ 실무 트러블슈팅 사례집 (Troubleshooting Log)

### ❌ Case 1: `Package has no installation candidate`
- **현상**: 패키지를 찾을 수 없어 설치 불가.
- **원인**: `/var/lib/apt/lists/` 내부의 메타데이터가 비어있음 (업데이트 안 함).
- **해결**: `sudo apt update` 명령어로 전단지 뭉치 생성.

### ❌ Case 2: `404 Not Found` (Failed to fetch)
- **현상**: 목록에는 존재하나 실제 다운로드 시 서버에 파일이 없음.
- **원인**: 내 로컬 정보는 구형인데, 원격 서버의 파일 버전이 업데이트됨 (불일치).
- **해결**: `sudo apt update`로 인덱스를 최신 서버 상태와 동기화.

### ❌ Case 3: `No package [name] available` (RedHat)
- **현상**: `yum install` 시 패키지가 없다고 나옴.
- **원인**: 입력한 이름이 정식 명칭이 아니거나 저장소가 활성화되지 않음.
- **해결**: `yum search [키워드]` 명령어로 서버가 인식하는 **'진짜 이름'** 확인 후 설치.

---

## 💡 Engineer's Insight
> "리눅스에서 패키지 설치 에러가 발생하면 무조건 **'apt update'** 혹은 **'yum clean all'**을 먼저 떠올리자. 로컬의 전단지와 서버의 실제 창고 상태를 맞추는 것이 해결의 시작이다."
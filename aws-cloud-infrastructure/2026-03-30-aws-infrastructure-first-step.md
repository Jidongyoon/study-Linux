## 📁 Repository Strategy: Structuring Cloud Assets
AWS 학습 및 프로젝트 관리를 위한 깃허브 디렉토리 체계 수립.

### 1. Selected Folder Name
- **Name**: `aws-cloud-infrastructure` (또는 마스터님이 선택하신 이름)
- **Purpose**: SAA 자격증 이론 및 실무 인프라(EC2, MariaDB) 설정값 영구 보존.

### 2. Management Rule
- **Standard**: 모든 파일명 및 폴더명은 소문자와 하이픈(-) 조합으로 통일하여 가독성 확보.
- **Content**: `README.md`를 최상단에 배치하여 프로젝트의 목적과 현재 진행 상황을 명시함.

> **Master's Strategy**: 
> "이름을 정하는 것은 프로젝트에 생명력을 불어넣는 작업이다. 체계적인 폴더 관리를 통해 87년생 늦깎이 공부가 아닌, 준비된 엔지니어의 기록임을 증명하겠다."

## 💰 Cost Management: Strategic Use of Free Tier Resources
AWS 프리티어(750시간/월)의 효율적 운영 및 비용 최적화 전략.

### 1. Free Tier Usage Pattern
- **24/7 Availability**: 월간 총 가용 시간(약 744시간)을 상회하는 750시간의 무료 제공량 확인.
- **Decision**: 학습의 연속성 및 퍼블릭 IP 유지(Persistence)를 위해 인스턴스를 상시 가동(Running) 상태로 유지함.

### 2. SAA Principle: Cost Awareness
- **Stopped State**: 인스턴스 중지 시 Compute 비용은 발생하지 않으나, 연결된 EBS(Storage) 비용은 발생할 수 있음을 인지.
- **Elastic IP Caveat**: 고정 IP 사용 시 인스턴스가 중지되면 오히려 비용이 청구되는 구조적 특성 파악.

> **Master's Efficiency**: 
> "무작정 아끼는 것이 최선은 아니다. 제공되는 자원을 최대한 활용하여 학습 효율을 극대화하는 것 또한 엔지니어의 자원 관리 전략이다. 현재 인스턴스는 수요일 프로젝트를 위해 'On' 상태를 유지한다."
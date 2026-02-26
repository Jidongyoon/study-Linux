Web Infrastructure & Cloud Service Implementation
본 문서는 Rocky Linux 9 환경에서 APM 스택을 활용하여 웹 어플리케이션(WordPress) 및 개인용 클라우드(ownCloud)를 구축한 과정을 개념적으로 정리한 보고서입니다.

1. 🏗️ The Foundation: APM Stack
웹 서비스를 구동하기 위한 가장 표준적이고 강력한 오픈소스 소프트웨어 조합입니다

구성 요소,기술 명칭,핵심 역할 (Role),비유

Apache,HTTP Web Server,클라이언트(브라우저)의 요청을 수신하고 응답을 반환하는 관문.,식당의 홀 직원

PHP,Scripting Language,사용자의 요청에 따라 동적인 로직을 처리하고 HTML을 생성함.,주방의 요리사

MariaDB,RDBMS,"모든 회원 정보, 게시글, 설정 데이터를 안전하게 보관하는 저장소.",식료품 창고

2. 📝 Content Management: WordPress
전 세계 웹사이트의 40% 이상이 사용하는 CMS(Content Management System)입니다.

Dynamic Content: 사용자가 코딩을 몰라도 웹상에서 글을 쓰고 관리할 수 있는 환경을 제공합니다.

Database Integration: 모든 포스팅과 사용자 설정은 실시간으로 MariaDB와 통신하여 저장됩니다.

Theme & Plugin: PHP 엔진을 기반으로 다양한 레이아웃과 기능을 확장할 수 있는 유연한 구조를 가집니다.

☁️ 3. Private Cloud: ownCloud
공용 클라우드(Google Drive, Dropbox)를 넘어 자신만의 독자적인 저장소 인프라를 구축한 사례입니다.

Self-Hosting: 데이터의 주권을 사용자가 직접 관리하는 서버에 두어 보안성과 독립성을 확보합니다.

Sync & Share: 윈도우/모바일 클라이언트와 서버 간의 실시간 동기화(Synchronize)를 지원합니다.

Storage Architecture:

/var/www/html/owncloud/data: 실제 데이터가 쌓이는 물리적 저장 경로.

Ownership: 웹 프로세스(apache)가 파일에 직접 접근할 수 있도록 권한을 할당받아 운영됩니다.

📡 4. Network Environment: Guest to Host
가상화 환경에서 서비스가 외부로 노출되고 연결되는 메커니즘입니다.

Virtual Networking:

IP Address: 192.168.111.100과 같은 사설 IP를 통해 내부 네트워크망 형성.

Port 80 (HTTP): 웹 서비스의 표준 통로로, 방화벽(firewalld) 설정을 통해 외부 접속을 허용함.

Cross-Platform Connectivity:

Host (Windows): 가상 머신 외부의 실제 컴퓨터에서 브라우저를 통해 서비스 호출.

Guest (Linux): 요청을 받아 실제 연산을 수행하고 결과를 반환하는 서버 역할.

🏆 5. Technical Insight: Summary
오늘의 실습을 통해 단순히 프로그램을 설치하는 것을 넘어 다음과 같은 기술적 정수를 습득하였습니다.

Encapsulation: 각 서비스(Apache, PHP, DB)가 독립적으로 작동하면서도 유기적으로 결합되는 과정을 이해함.

Infrastructure as a Service: 물리적인 서버 없이도 가상화 기술을 통해 완벽한 웹 서비스를 배포할 수 있음을 확인함.

Data Sovereignty: 클라우드 솔루션 구축을 통해 데이터의 흐름과 저장 방식에 대한 관리적 역량을 배양함.
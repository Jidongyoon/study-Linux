Troubleshooting: External Access to Internal Web Server via DNAT
본 문서는 VMware 가상화 환경에서 방화벽 서버(Gateway)를 통해 사설망 내부의 웹 서버에 외부 클라이언트가 접속하는 과정에서 발생한 트러블슈팅 사례를 기록합니다.

🏗️ 1. 네트워크 구성도 (Network Topology)
External Client (WinClient): 192.168.111.128 (VMnet8 NAT)

Firewall/Gateway (Server A):

External: 192.168.111.100 (ens160)

Internal: 10.1.1.1 (ens224)

Web Server (Server B): 10.1.1.20 (ens160 / Gateway: 10.1.1.1)

🛠️ 2. 트러블슈팅 로그 (Troubleshooting Process)
❌ Issue 1: Destination Host Unreachable (L2/L3 단절)
현상: 클라이언트에서 게이트웨이(10.1.1.1)로 핑이 가지 않음.

원인 파악:

nmcli device status 확인 결과, 하드웨어는 연결되었으나 IP 대역이 설계도와 다름.

VMware의 **VMnet 설정(가상 스위치)**이 서로 다른 구멍에 꽂혀 있어 물리적 통신 불가.

해결:

VMware 설정에서 두 서버의 내부망 어댑터를 동일한 **VMnet0**으로 통일.

nmcli con mod를 통해 ens224 인터페이스에 고정 IP(10.1.1.1/24) 강제 할당.

❌ Issue 2: DHCP 할당 실패 및 APIPA 발생
현상: 외부 클라이언트(WinClient)의 IP가 169.254.x.x로 할당되어 방화벽 서버(192.168.111.100)와 통신 불가.

원인 파악: 강의 초반 설정한 VMware DHCP 서비스가 비활성화되어 윈도우가 임의의 주소(APIPA)를 부여함.

해결:

윈도우 네트워크 설정에서 **Static IP(192.168.111.128)**를 수동으로 지정하여 방화벽 서버와 동일 대역(L3) 확보.

❌ Issue 3: DNAT 설정 후 웹 접속 불가 (응답 경로 누락)
현상: iptables로 포트 포워딩을 설정했으나, 브라우저에서 '사이트에 연결할 수 없음' 발생.

원인 파악:

Forwarding 비활성화: 리눅스 커널이 패킷을 넘겨주는 기능(ip_forward)이 꺼져 있음.

Return Path 누락: 내부 서버(B)가 응답을 보낼 때 외부로 나가는 문(Masquerade) 설정 미흡.

해결:

sysctl -w net.ipv4.ip_forward=1 설정으로 커널 라우팅 활성화.

iptables -t nat -A POSTROUTING -j MASQUERADE 추가하여 응답 패킷의 경로 확보.

iptables -P FORWARD ACCEPT로 패킷 통과 허용.

✅ 3. 최종 성공 확인 (Final Verification)
🖥️ 적용된 핵심 커맨드 (방화벽 서버)

# 1. IP 포워딩 활성화
sysctl -w net.ipv4.ip_forward=1

# 2. DNAT (포트 포워딩 80 -> 10.1.1.20:80)
iptables -t nat -A PREROUTING -p tcp -i ens160 --dport 80 -j DNAT --to-destination 10.1.1.20

# 3. Masquerade (응답 패킷 변환)
iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE

# 4. Forwarding 허용
iptables -I FORWARD -j ACCEPT

결과 분석
브라우저 경고 ("안전하지 않음"): HTTP 통신 및 SSL 인증서 미적용에 따른 정상적인 경고 메시지임. 이를 통해 방화벽을 거쳐 내부 웹 서버까지 TCP 연결이 성사되었음을 확인.

콘텐츠 확인: 웹 서버(B)의 /var/www/html/index.html 파일을 수정하여 윈도우 브라우저에 정상적으로 텍스트가 출력되는 것을 최종 확인.

💡 4. Lessons Learned (배운 점)
계층적 접근: 네트워크 문제는 항상 물리 계층(VMnet) -> 데이터 링크/네트워크 계층(IP/Subnet) -> 전송 계층(Port) 순으로 점검해야 함을 체득함.

NAT의 양방향성: 들어오는 길(DNAT)만 열어주는 것이 아니라, 나가는 길(Masquerade/Gateway)이 정확해야 완벽한 통신이 이루어짐을 이해함.

로그의 중요성: ip addr, ip route, iptables -L 등 시스템 로그와 상태 확인 명령어가 트러블슈팅의 핵심임을 깨달음.

가상 네트워크 환경의 제약 사항 및 구조
실습 환경: VMware Workstation Pro

네트워크 개념:

VMnet8 (NAT): 본 실습에서 '공인 인터넷(외부망)' 역할을 수행합니다.

VMnet0 (Bridge/Custom): 본 실습에서 외부와 격리된 '기업 내부 사설망' 역할을 수행합니다.

접속 규칙 (Access Rule):

가상 환경의 한계: 외부 클라이언트는 반드시 가상 게이트웨이가 관리하는 대역(192.168.111.x) 내에 물리적으로 연결되어 있어야만 통신이 가능합니다. (L2/L3 인접성 필요)

실제 환경과의 차이: 실제 운영 환경에서는 ISP(통신사)가 전 세계적인 라우팅 경로를 동기화하므로, 클라이언트가 어떤 대역에 있더라도 공인 IP를 통해 서버에 접속할 수 있습니다.

결론: 본 트러블슈팅 사례는 이러한 가상화 환경의 특성을 고려하여, 클라이언트의 IP 대역을 일치시킨 후 방화벽의 DNAT 정책을 검증하는 것에 초점을 맞추었습니다.
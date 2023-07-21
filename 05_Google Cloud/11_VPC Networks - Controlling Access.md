
두 개의 웹 서버(nginx) 생성 후 태그가 지정된 방화벽 규칙을 사용하여 웹 서버에 대한 외부 HTTP 액세스 제어 및 IAM 역할과 서비스 계정 탐색

### 웹 서버 생성

웹 서버 생성 및 nginx 설치 후 시작 페이지를 수정하여 서버 구분
1. server1 생성
	Google Cloud Console > Compute Engine > VM Instance > CREATE VM INSTANCE
	이름: server1
	고급 옵션 > 네트워킹 > 네트워크 태그: web-server
2. server2 생성
	변경 사항 없이 기본 값으로 설정

#### nginx 설치 및 시작 페이지 사용자 지정

두 VM 인스턴스에 nginx를 설치하고 시작 페이지를 수정하여 서버 구분
1. VM 인스턴스에서 server1의 SSH를 클릭하여 터미널 시작 및 연결
2. server1의 SSH에서 nginx 설치
	- sudo apt-get install nginx-light -y
3. nano 편집기에서 시작 페이지 열기
	- sudo nano /var/www/html/index.nginx-debian.html
4. 편집기에서 아래 내용 수정
	- 변경 전: <h1>Welcome to nginx!</h1>
	- 변경 후: <h1>Welcome to the server1!</h1>
	Ctrl + O > ENTER > Ctrl + X
5. 변경 사항 확인
	- cat /var/www/html/index.nginx-debian.html
6. server2에도 위 과정 반복
	*4번 내용만 변경 후에서 server2로만 변경할 것
7. SSH 터미널 종료
	- exit

### 방화벽 규칙 만들기

태그가 지정된 방화벽 규칙 생헝 후 HTTP 연결 테스트

#### 태그가 지정된 방화벽 규칙 만들기

웹 서버 네트워크 태그를 사용하여 VM 인스턴스에 적용되는 방화벽 규칙 만들기
1. Google Cloud Console > Menu > VPC Network > Firewall
2. default-allow-internal 방화벽 규칙 확인
	*default-allow-internal 방화벽 규칙은 기본 네트워크 내의 모든 프로토콜/포트에서 트래픽을 허용한다.
	다음 단계에서 네트워크 태그(web-server)를 사용하여 네트워크 외부에서 server1으로만 트래픽을 허용하는 방화벽 규칙을 생성할 것
3. CREATE FIREWALL RULES
	example)
	이름: allow-http-web-server
	네트워크: default
	타겟: Specified target tags
	타겟 태그: web-server
	Source filter: IPv4 Ranges
	Source IPv4 ranges: 0.0.0.0/0
	Protocols and ports: Specified protocols and ports > Check tcp & type: 80 > Check Other & type: icmp

#### HTTP 연결 테스트

1. Google Cloud Console > Menu > Compute Engine > VM Instances
2. server1 및 server2의 내부 및 외부 IP 주소 기록
3. VM 인스턴스에서 test-vm의 SSH에 접속하여 터미널 시작
4. server1의 내부 IP에 대한 HTTP 연결을 테스트
	- curl `server1's internal IP`
5. server2의 내부 IP에 대한 HTTP 연결을 테스트
	- curl -c 3 `server2's internal IP`
	*내부 IP 주소를 사용하여 두 서버 모두에 HTTP 액세스를 할 수 있다.
	test-vm은 웹 서버 기본 네트워크와 동일한 VPC 네트워크에 있기 때문에 tcp:80의 연결은 default-allow-internal 방화벽 규칙에 의해 허용된다.
6. server1의 외부 IP에 대한 HTTP 연결을 테스트
	- curl `server1's external IP`
7. server2의 외부 IP에 대한 HTTP 연결을 테스트
	- curl -c 3 `server2's external IP`
	*위 명령은 작동하지 않기 때문에 Ctrl + C를 통해 HTTP 요청 중지
	웹 서버 태그가 있는 VM 인스턴스에만 allow-http-web-server가 적용되므로 server1의 외부 IP 주소에만 HTTP 액세스가 가능하다.

### 네트워크 및 보안 관리자 역할

Cloud IAM을 사용하면 특정 리소스에 대해 작업을 수행할 수 있는 권한을 부여하여 클라우드 리소스를 중앙에서 관리할 수 있는 전체 제어 및 가시성을 얻을 수 있다.

네트워크 관리자와 보안 관리자는 단일 프로젝트 네트워킹과 함께 사용되어 각 VPC 네트워크에 대한 관리 액세스를 독립적으로 제어한다.
	1. 네트워크 관리자
	방화벽 규칙 및 SSL 인증서를 제외한 네트워킹 리소스를 생성, 수정 및 삭제할 수 있는 권한
	2. 보안 관리자
	방화벽 규칙 및 SSL 인증서 생성, 수정 및 삭제할 수 있는 권한

개별 사용자 대신 VM 인스턴스에 속하는 특별한 Google 계정인 서비스 계정에 역할을 적용한다.
새 사용자를 생성하는 대신 서비스 계정을 사용하여 네트워크 관리자 및 보안 관리자 역할의 권한을 입증하도록 test-vm에 권한을 부여한다.

#### 현재 권한 확인

1. VM 인스턴스에서 test-vm의 SSH 터미널에 접속
2. 방화벽 규칙 리스트 확인
	- gcloud compute firewall-rules list
	*네트워크 관리자 역할에는 방화벽 규칙을 나열할 수 있지만 수정 및 삭제할 수 있는 권한이 없으므로 작동하지 않는다.

#### 서비스 계정 업데이트 및 권한 확인

보안 관리자 역할을 제공하여 네트워크 관리자 서비스 계정을 업데이트

1. IAM에서 네트워크 관리자 계정 수정
	역할을 Compute Engine > Compute Security Admin으로 수정 및 저장
2. test-vm 인스턴스의 SSH 터미널로 돌아가 방화벽 규칙 리스트 확인
	- gcloud compute firewall-rules list
3. allow-http-web-server 방화벽 규칙 삭제
	- gcloud compute firewall-rules delete allow-http-web-server
	*위 과정을 통해 Security Admin 역할에 방화벽 규칙을 나열하고 삭제할 수 있는 권한이 있음을 알 수 있다.

#### 방화벽 규칙 삭제 확인

1. test-vm 인스턴스의 SSH 터미널에 접속
2. server1의 외부 IP에 대한 HTTP 연결을 테스트
	- curl -c 3 `server1's external IP`
	*allow-http-web-server 방화벽 규칙을 삭제했기 때문에 더 이상 server1의 외부 IP에 HTTP 액세스를 할 수 없으므로 작동하지 않는다.
3. HTTP 요청 중지
	- Ctrl + C
	*방화벽 규칙에 대한 원치 않는 변경을 방지하려면 올바른 사용자 또는 서비스 계정에 보안 관리자 역할을 제공해야 한다.
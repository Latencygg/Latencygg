
Google Cloud Virtual Private Cloud(VPC)는 Compute Engine 가상 머신 인스턴스, Kubernetes Engine 컨테이너 및 App Engine Flex에 네트워킹 기능을 제공한다.
즉, VPC 네트워크가 없으면 VM 인스턴스, 컨테이너 또는 App Engine 애플리케이션을 만들 수 없다.
따라서 각 Google Cloud 프로젝트에는 시작할 수 있는 기본 네트워크가 있다.

VPC 네트워크는 Google Cloud 내에서 가상화된다는 점을 제외하면 물리적 네트워크와 동일한 방식으로 생각할 수 있다.
VPC 네트워크는 데이터 센터의 지역 가상 서브네트워크(서브넷) 목록으로 구성된 글로벌 리소스이며 모두 글로벌 광역 네트워크(WAN)로 연결된다.
VPC 네트워크는 Google Cloud에서 서로 논리적으로 격리된다.

### 기본 네트워크 탐색

Google Cloud 프로젝트에는 서브넷, 경로, 방화벽 규칙이 있는 기본 네트워크가 있다.

#### 서브넷(Sub Network)

기본 네트워크에는 Google Cloud 리전에 서브넷이 있다.

1. Google Cloud Console > VPC Network > VPC Network
2. 기본 네트워크를 클릭하여 기본 네트워크 세부 정보 및 서브넷 확인

#### 경로(Route)

경로는 인스턴스에서 네트워크 내부 또는 Google Cloud 외부의 목적지로 트래픽을 보내는 방법을 VM 인스턴스와 VPC 네트워크에 알려준다.
각 VPC 네트워크에는 서브넷 간에 트래픽을 라우팅하고 적합한 인스턴스에서 인터넷으로 트래픽을 보내는 몇 가지 기본 경로가 있다.

1. Google Cloud Console > VPC Network > Route
2. Effective Routes에서 default 네트워크와 리전 선택
	*각 서브넷에 대한 경로가 있고 기본 인터넷 게이트웨이에 대한 경로가 있음

#### 방화벽 규칙 보기

각 VPC 네트워크는 사용자가 구성할 수 있는 분산 가상 방화벽을 구현한다.
방화벽 규칙을 사용하면 어떤 패킷이 어떤 목적지로 이동할 수 있는지 제어할 수 있다.
모든 VPC 네트워크에는 모든 수신 연결을 차단하고 모든 발신 연결을 허용하는 두 가지 묵시적 방화벽 규칙(Implied Firewall Rules)이 있다.

1. Google Cloud Console > VPC Network > Firewall
2. 기본 네트워크에 대한 4개의 Ingress 방화벽 규칙이 있다
	default-allow-icmp, default-allow-internal, default-allow-rdp, default-allow-ssh

#### 기본 네트워크 삭제 및 VM 인스턴스 생성 시도

1. 모든 방화벽 규칙을 선택하고 DELETE
2. VPC Network에서 기본 네트워크를 클릭하여 VPC 네트워크 삭제 후 Route에서 확인
	*VPC 네트워크가 없으면 경로가 없음
3. Google Cloud Console > Compute Engine > VM Instance
4. 모든 값을 기본값으로 두고 VM 인스턴스 생성
	*고급 옵션 섹션 > 네트워크 인터페이스를 확인해보면 사용 가능한 VPC Network가 없기 때문에 VM 인스턴스를 생성할 수 없다는 것을 알 수 있음

### VPC 네트워크 및 VM 인스턴스 만들기

VM 인스턴스를 생성하기 위해 VPC Network 생성

#### 방화벽 규칙을 사용하여 자동 모드 VPC 네트워크 만들기

1. Google Cloud Console > VPC Network > CREATE VPC NETWORK
	이름 설정, 서브넷 생성 모드: 자동

#### VM 인스턴스 생성 지역

리전 및 영역을 선택하면 서브넷이 결정되고 서브넷의 IP 주소 범위에서 내부 IP 주소가 할당된다.

1. Google Cloud Console > Compute Engine > VM Instance > CREATE INSTANCE
	리전 및 영역이 다른 2개의 인스턴스 생성

### VM 인스턴스에 대한 연결 탐색

VM 인스턴스에 대한 연결을 탐색한다.
	tcp:22를 사용하여 VM 인스턴스에 SSH로 연결하고 ICMP를 사용하여 VM 인스턴스 내부 및 외부 IP 주소를 모두 핑한다. 이후 방화벽 규칙을 하나씩 제거하여 방화벽 규칙이 연결에 미치는 영향 확인

1. 하나의 인스턴스의 내부 및 외부 IP 주소 기록
2. 해당 인스턴스의 SSH를 시작하여 터미널에 연결
	tcp:22에 대해 어디서나(0.0.0.0/0) 들어오는 트래픽을 허용하는 allow-ssh 방화벽 규칙에 의해 SSH를 사용할 수 있다.
3. 내부 IP에 대한 연결 테스트
	- ping -c 3 `internal IP about instance1`
	*allow-custom 방화벽 규칙 때문에 내부 IP를 핑할 수 있다.
4. 외부 IP에 대한 연결 테스트
	- ping -c 3 `external IP about instance1`

### allow-icmp 방화벽 규칙 제거

1. allow-icmp 방화벽 규칙 제거 후 인스턴스의 내부 및 외부 IP 주소에 대해 ping 시도
	1) Google Cloud Console > VPC Network > Firewall
	2) `instance name`-allow-icmp 삭제
	3) 전에 생성했던 다른 인스턴스(`instance2`)의 SSH에서 현재 삭제한 인스턴스의 내부 IP에 대한 연결 테스트
		- ping -c 3 `internal IP about instance1`
			*allow-custom 방화벽 규칙이 존재하기 때문에 내부 IP를 핑할 수 있다.
	4) 삭제한 인스턴스의 외부 IP에 대한 연결 테스트
		- ping -c 3 `external IP about instance1`
	*100% 패킷 손실은 삭제한 인스턴스(allow-icmp 방화벽 규칙 삭제)의 외부 IP에 핑할 수 없음을 나타낸다.

### allow-custom 방화벽 규칙 제거

1. allow-custom 방화벽 규칙 제거 후 인스턴스의 내부 IP 주소에 대해 ping 시도
	*100% 패킷 손실은 삭제한 인스턴스(allow-custom 방화벽 규칙 삭제)의 내부 IP에 핑할 수 없음을 나타낸다.
2. SSH 터미널 종료
	- exit

### allow-ssh 방화벽 규칙 제거

1. allow-ssh 방화벽 규칙을 제거한 후 `instance2`에서 SSH 시도
	*연결 실패 메세지는 allow-ssh 방화벽 규칙을 삭제했기 때문에 `instance2`의 SSH로 연결할 수 없음을 나타낸다.



### 개요

Google Cloud Virtual Private Cloud(VPC) 네트워크 피어링을 사용하면 두 VPC 네트워크가 동일한 프로젝트 또는 동일한 조직에 속하는지 여부에 관계없이 비공개 연결이 가능하다.
VPC 네트워크 피어링은 Google Cloud SaaS(Software-as-a-Service) 생태계를 구축하여 조직 내외의 다양한 VPC 네트워크에서 비공개로 서비스를 사용할 수 있으므로 워크로드가 비공개 공간에서 통신할 수 있다.

VPC 네트워크 피어링은 여러 네트워크 관리 도메인이 있거나 다른 조직과 피어링하려는 조직에서 사용할 때 유용하다.
조직 내에 여러 네트워크 관리 도메인이 있는 경우 VPC 네트워크 피어링을 사용하면 비공개 공간의 VPC 네트워크에서 서비스를 사용할 수 있다.
다른 조직에 서비스를 제공하는 경우 VPC 네트워크 피어링을 사용하면 해당 조직의 비공개 공간에서 해당 서비스를 사용할 수 있다.

조직 전체에서 서비스를 제공하는 기능은 다른 기업에 서비스를 제공하려는 경우에 유용하며, 자체 구조로 인해 혹은 인수합병의 결과로 여러 개의 고유한 조직 노드가 있는 경우 자체 기업 내에서 유용하다.

VPC 네트워크 피어링은 외부 IP 주소 또는 VPN을 사용하여 네트워크를 연결하는 것보다 다음과 같은 몇 가지 이점을 제공한다.
1. 네트워크 대기 시간
	사설 네트워킹은 공용 IP 네트워킹보다 대기 시간이 짧다.
2. 네트워크 보안
	서비스 소유자는 공용 인터넷에 서비스를 노출하고 관련 위험을 처리할 필요가 없다.
3. 네트워크 비용
	피어링된 네트워크는 내부 IP를 사용하여 통신하고 Google Cloud 이그레스 대역폭 비용을 절약할 수 있다. 일반 네트워크 가격은 여전히 모든 트래픽에 적용된다.

### VPC 네트워크 피어링 설정

동일한 조직 노드 내에서 네트워크는 동일하거나 다른 프로젝트의 다른 VPC 네트워크에서 액세스할 수 있어야 하는 서비스를 호스팅할 수 있고, 또는 한 조직에서 타사 서비스가 제공하는 서비스에 액세스하려고 할 수 있다.

프로젝트 이름은 모든 Google Cloud에서 고유하므로 피어링을 설정할 때 조직을 지정할 필요가 없다. Google Cloud는 프로젝트 이름을 기반으로 조직을 알고 있다.

#### 프로젝트에서 사용자 정의 네트워크 생성

example) 프로젝트 2개 프로비저닝(project-A, project-B)
1. Cloud Shell을 실행하여 두 개의 프로젝트를 관리하기 위해 새로운 탭(+)을 클릭하여 새 Cloud Shell 실행
2. 두 번째 Cloud Shell에서 두 번째 프로젝트의 프로젝트 ID 설정
	- gcloud config set project

[첫 번째 Cloud Shell]
3. 커스텀 네트워크를 생성하고 실행
	- gcloud compute networks create network-a --subnet-mode custom
	*network-a: project-A의 `network name`
4. 서브넷 생성 후 리전 및 IP 범위 지정
	- gcloud compute networks subnets create network-b-subnet --network network-b \ --range 10.8.0.0/16 --region 
	*network-b: project-B의 `network name`
5. VM 인스턴스 생성
	- gcloud compute instances create vm-a --zone --network network-a --subnet network-a-subnet --manchine-type e2-small
	*vm-a: project-A의 `vm instance name`
6. VM과 통신하기 위해 Secure Shell이 필요하여 SSH 및 icmp 활성화 및 실행
	- gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp

[두 번째 Cloud Shell]
7. 커스텀 네트워크 생성
	- gcloud compute networks create network-b --subnet-mode custom
8. 서브넷 생성 후 리전 및 IP 범위 지정
	- gcloud compute networks subnets create network-b-subnet --network network-b \ --range 10.8.0.0/16 --region
9. VM 인스턴스 생성
	- gcloud compute instances create vm-b --zone  --network network-b --subnet network-b-subnet --machine-type e2-small
10. VM과 통신하기 위해 Secure Shell이 필요하여 SSH 및 icmp 활성화 및 실행
	- gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp

### VPC 네트워크 피어링 세션 설정

project-A의 network-a, project-B의 network-b 간에 VPC 네트워크 피어링이 성공적으로 설정되려면 network-a와 network-b의 관리자가 별도로 피어링 연결을 구성해야 한다.

[project-A]
1. Google Cloud Console > VPC Network > VPC Network Peering
2. example)
	1) 이름: peer-ab
	2) 피어링하려는 네트워크: network-a
	3) In another project 체크 및 두 번째 프로젝트의 프로젝트 ID 입력: {{{ project_1.project_id }}}
	4) 다른 네트워크(network-b)의 VPC 네트워크 이름 입력
	*해당 시점에서 project-B의 network-b에 일치하는 구성이 없기 때문에 피어링 상태는 INACTIVE로 유지된다(Waiting for peer network to connect).

[project-B]
*콘솔에서 두 번째 프로젝트(project-B)로 전환할 것
1. example)
	1) 이름: peer-ba
	2) 피어링하려는 네트워크: network-b
	3) In another project 체크 및 두 번째 프로젝트의 프로젝트 ID 입력: {{{ project_0.project_id }}}
	4) 다른 네트워크(network-a)의 VPC 네트워크 이름 입력
	*VPC 네트워크 피어링이 ACTIVE가 되고 경로가 교환되며, 피어링이 ACTIVE 상태로 이동하는 즉시 트래픽 흐름이 설정된다.
		피어링된 네트워크의 VM 인스턴스 간에 풀 메시 연결이 되어있음
		네트워크의 VM 인스턴스로부터 피어링된 네트워크에서의 내부 부하 분산 엔드포인트로 이동
	*피어링된 네트워크 CIDR에 대한 경로는 VPC 네트워크 피어 전체에서 확인할 수 있고, 이러한 경로는 활성화된 피어링에 대해 생성된 경로이다.
		- gcloud compute routes list --project `project ID of project-A`
			*해당 명령어로 project-A의 모든 VPC 네트워크 경로를 나열할 수 있다.

### 연결 테스트

[project-A]
1. Google Cloud Console > Menu > Compute Engine > VM Instance
2. vm-a의 INTERNAL_IP 복사

[project-B]
1. Google Cloud Console > Menu > Compute Engine > VM Instance
2. vm-b의 SSH를 통해 인스턴스에 연결
3. vm-a의 인스턴스 INTERNAL_IP를 입력하여 네트워크 피어링 연결이 되었는지 확인
	- ping -c 5 `internal IP of vm-a`
Cloud Shell에서는 명령줄을 사용하여 Google Cloud에 호스팅된 컴퓨팅 리소스에 엑세스할 수 있다.
Cloud Shell은 영구적인 5GB 홈 디렉터리를 갖춘 Debian 기반의 가상 머신으로, Google Cloud 프로젝트 및 리소스를 쉽게 관리할 수 있게 한다.
Cloud Shell에는 'gcloud' 명령줄 도구와 필요한 기타 유틸리티가 사전 설치되어 있어 프로젝트 및 리소스를 빠르게 준비하고 실행할 수 있다.

### 리전 및 영역 설정

1. 프로젝트 리전 설정
	- gcloud config set compute/region `region name`
	example) gcloud config set compute/region us-central1
2. 프로젝트 리전 설정 보기
	- gcloud config get(-value) compute/region
3. 영역(존) 설정
	- gcloud config set compute/zone `zone name`
	example) gcloud config set compute/zone us-central1-a
4. 영역(존) 설정 보기
	- gcloud config get(-value) compute/zone

### 프로젝트 정보 확인하기

1. Cloud Shell에서 프로젝트의 프로젝트 ID를 보기
	- gcloud config get-value project
2. Cloud Shell에서 프로젝트 세부정보 보기
	- gcloud compute project-info describe --project $(gcloud config get-value project)

### 환경 변수 설정

환경을 정의하는 환경 변수는 API 또는 실행 파일이 포함된 스크립트를 작성할 때 시간을 절약하는데 도움이 된다.

1. 프로젝트 ID를 저장할 환경 변수를 만든다.
	- export PROJECT_ID=$(gcloud config get-value project)
2. 영역을 저장할 환경 변수를 만든다.
	- export ZONE=$(gcloud config get-value compute/zone)
3. 변수가 적절하게 설정되었는지 확인한다.
	- echo -e "PROJECT ID: $PROJECT_ID \n ZONE: $ZONE"
*변수가 올바르게 설정된 경우 echo 명령어는 프로젝트 ID와 영역을 출력한다.

### gcloud 도구로 가상머신 만들기

gcloud 도구를 사용하여 새 가상 머신(VM) 인스턴스를 만든다.

1. VM 만들기
	- gcloud compute instances create `VM instance name` --machine-type `machine type` --zone `zone name`
	example) gcloud compute instances create gcelab --machine-type e2-medium --zone $ZONE
	
	[명령어 세부 정보]
	1. gcloud compute
		Compute Engine API보다 간단한 형식으로 Compute Engine 리소스를 관리한다.
	2. instances create
		새 인스턴스 생성
	3. `VM instance name`
		가상 머신(VM)의 이름
	4. --machine-type
		머신 타입 플래그는 머신 유형을 `machine type`으로 지정해준다.
	5. --zone
		영역 플래그는 VM이 생성되는 위치를 지정하고, 영역 플래그를 생략하게 되면 gcloud 도구가 기본 속성을 기준으로 개발자가 원하는 영역을 추론할 수 있다. machine type 및 image와 같은 기타 필수 인스턴스 설정은 create 명령어에서 지정되지 않은 경우 기본값으로 설정된다.

### gcloud 명령어

1. gcloud 도구에서는 gcloud 명령어 후에 -h 플래그를 추가하면 참고할 수 있는 간단한 사용 가이드 라인을 제공한다
	- gcloud -h
	- gcloud config --help
	- gcloud help config
2. 환경에서 구성 목록 보기
	- gcloud config list
3. 모든 속성과 각 설정 보기
	- gcloud config list -all
4. 구성요소 나열
	- gcloud components list

### 명령줄 출력 필터링하기

gcloud CLI는 명령줄에서 작동하는 유용한 도구이며, 특정 정보를 표시해야 하는 경우가 있다.

1. 프로젝트에서 이용 가능한 컴퓨팅 인스턴스를 나열한다.
	- gcloud compute instances list
2. VM 인스턴스 가상 머신 나열
	- gcloud compute instances list --filter="name=('`VM instance name`')"
3. 프로젝트의 방화벽 규칙 나열
	- gcloud compute firewall-rules list
4. 기본 네트워크의 방화벽 규칙 나열
	- gcloud compute firewall-rules list --filter="network='default'"
5. 허용 규칙이 ICMP 규칙과 일치하는 기본 네트워크의 방화벽 규칙 나열
	- gcloud compute firewall-rules list --filter="NETWORK:'default' AND ALLOW:'ICMP'"

### VM 인스턴스에 연결하기

gcloud compute를 사용하면 인스턴스에 쉽게 연결할 수 있다.
gcloud compute ssh 명령어는 SSH에 래퍼 기능을 제공하여 인증 및 인스턴스 이름과 IP 주소의 매핑을 처리하게 된다.

1. SSH를 사용하여 VM 인스턴스에 연결
	- gcloud compute ssh `VM instance name` --zone `zone name`
	*후에 Yes 및 비밀번호 등록하는 부분에서 Enter 2번 입력
2. 가상 머신에 nginx 웹 서버 설치
	- sudo apt install -y nginx
3. SSH 연결을 끊고 원격 셸 종료
	- exit

### 방화벽 업데이트하기

가상 머신과 같은 컴퓨팅 리소스를 사용하는 경우 연결된 방화벽 규칙을 파악해야 한다.

1. 프로젝트의 방화벽 규칙 나열
	- gcloud compute firewall-rules list
	*위의 출력에서 사용 가능한 2개의 네트워크를 확인할 수 있고, default 네트워크는 가상 머신이 있는 위치이다.
2. 가상 머신에서 실행 중인 nginx 서비스에 엑세스를 시도한다.
	*적절한 방화벽 규칙이 없기 때문에 가상 머신과의 통신에 실패하게 되는데, 웹서버 통신이 작동하기 위해 가상 머신에 태그를 추가하고, http 트래픽에 대한 방화벽 규칙을 추가한다.
3. 가상 머신에 태그 추가
	- gcloud compute instances add-tags `VM instance name` --tags http-server, https-server
4. 허용할 방화벽 규칙 업데이트
	- gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
5. 프로젝트 방화벽 규칙 나열
	- gcloud compute firewall-rules list --filter=ALLOW:'80'
6. 가상 머신에 대한 http 통신이 가능한지 확인
	- curl http://$(gcloud compute instances list --filter=name:`VM instance name` --format='value(EXTERNAL_IP)'

### 시스템 로그 보기

로그 보기는 프로젝트의 작업을 이해하는 데 필수적이다.
gcloud를 사용해 Google Cloud에서 사용 가능한 다양한 로그에 엑세스할 수 있다.

1. 시스템에서 사용 가능한 로그 확인
	- gcloud logging logs list
2. 컴퓨팅 리소스와 관련된 로그 보기
	- gcloud logging logs list --filter="compute"
3. `instance` 리소스 유형과 관련된 로그 읽기
	- gcloud logging read "resource.type=`instance`" --limit 5
4. 특정 가상 머신의 로그 읽기
	- gcloud logging read "resource.type=`instance` AND labels.instance_name='`VM instance name`'" --limit 5







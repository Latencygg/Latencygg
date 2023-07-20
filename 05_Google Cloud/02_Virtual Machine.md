Compute Engine을 사용하면 Google의 인프라상에서 다양한 Linux(Debian, Ubuntu 등), Windows Server 등 서로 다른 운영체제가 실행되는 가상 머신을 만들 수 있다.
속도가 빠르고 일관적인 성능을 제공하도록 설계된 시스템에서 수천 개의 가상 CPU를 실행할 수 있다.

### 리전 및 영역의 이해

일부 Compute Engine 리소스는 리전(Region)이나 영역(Zone)에 있다.
리전은 리소스를 실행할 수 있는 특정 지리적 위치이며, 각 리전에는 하나 이상의 영역이 있다.
예를 들어 'us-central1' 리전은 'us-central1-a', 'us-central1-b', 'us-central1-c' 및 'us-central1-f' 영역이 있는 미국 중부의 리전을 나타낸다.

영역 내에 상주하는 리소스를 영역별 리소스라고 한다.
가상 머신 인스턴스와 영구 디스크는 영역에 상주하고, 영구 디스크를 가상 머신 인스턴스에 연결하려면 두 리소스가 모두 같은 영역에 있어야 한다.
마찬가지로 인스턴스에 정적 IP 주소를 할당하려는 경우 인스턴스가 정적 IP와 같은 리전에 있어야 한다.

### Cloud 콘솔에서 새로운 인스턴스 만들기

1. 사용 중인 계정 이름 목록 표시
	- gcloud auth list
2. 프로젝트 ID 목록 표시
	- gcloud config list project

Cloud 콘솔에서 Compute Engine을 사용하여 사전 정의된 머신 유형을 새로 만들 수 있다.

1. Cloud 콘솔의 탐색 메뉴 -> VM 인스턴스 -> 인스턴스 만들기
	새 인스턴스를 만들 때 다양한 매개변수를 구성할 수 있다.
	가상 머신이 생성되면 VM 인스턴스 페이지에 나열된다.
2. 가상 머신을 원하는 인스턴스에 연결하기 위해 많이 사용되어지는 웹 서버 중 하나인 NGINX 웹 서버 설치
	OS 업데이트
		- sudo apt-get update
	NGINX 설치
		- sudo apt-get install -y nginx
	NGINX가 실행 중인지 확인
		- ps auwx | grep nginx
	
	*웹페이지를 보려면 Cloud 콘솔로 돌아와 머신이 표시된 행에서 외부 IP 링크를 클릭하거나, 새 브라우저 창 또는 탭에서 외부 IP 값을 'http://EXTERNAL_IP' 에 추가한다.

### gcloud로 새 인스턴스 만들기

Cloud 콘솔을 사용하여 가상 머신 인스턴스를 만드는 대신, Google Cloud Shell에 사전 설치되어 있는 명령 도구 'gcloud'를 사용할 수 있다.
Cloud Shell은 필요한 모든 개발 도구가 로드된 Debian 기반 가상 머신으로, 5GB의 영구적인 홈 디렉토리를 제공한다.

1. Cloud Shell에서 'gcloud'를 사용해 명령줄에서 새 가상 머신 인스턴스를 만든다.
	- gcloud compute instances create gcelab --machine-type e2-medium --zone zone name
2. 모든 기본값 보기
	- gcloud compute instances create --help
3. help 종료
	- Ctrl + C

*항상 하나의 리전 또는 영역내에서 작업하며 매번 --zone 플래그를 추가하고 싶지 않다면 gcloud에서 사용할 기본 리전과 영역을 설정할 수 있다.
1. 리전 설정
	- gcloud config set compute/region ..
2. 존 설정
	- gcloud config set compute/zone ..

또한 SSH를 사용하여 gcloud를 통해 인스턴스에 연결할 수 있다.
이 경우 영역을 추가해야 하며, 옵션을 전역으로 설정한 경우에는 --zone 플래그를 생략해야 한다.
	- gcloud compute ssh gcelab --zone
	*암호 섹션에서 Enter를 눌러 암호 입력을 생략할 수 있고, 연결 후에는 exit를 통해 원격 셀을 종료하여 SSH 연결을 끊는다. 
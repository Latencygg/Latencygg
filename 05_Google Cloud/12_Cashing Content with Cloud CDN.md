
### 개요

Cloud CDN은 전 세계에 분산된 Google의 엣지 접속 지점을 사용하여 사용자와 가까운 HTTP(S) 부하 분산 컨텐츠를 캐시한다.
Google 네트워크의 가장자리에 컨텐츠를 캐싱하면 서비스 비용을 줄이면서 사용자에게 컨텐츠를 더 빠르게 전달할 수 있다.

### Cloud Storage 버킷 생성 및 채우기

Cloud CDN 컨텐츠는 두 가지 유형의 백엔드에서 생성될 수 있다.
	1. Google Compute Engine VM 인스턴스 그룹
	2. Google Cloud Storage 버킷

#### 고유한 Cloud Storage 버킷 만들기

1. Google Cloud Console > Menu > Cloud Storage > Bucket > CREATE BUCKET
2. 버킷의 이름을 고유한 값으로 입력 한 후 리전 및 영역 선택
	*실습 시 Location의 경우 지구 반대편에 있거나 최소한 다른 대륙을 선택한다.
	Cloud CDN을 사용 설정한 상태와 사용하지 않은 상태에서 이미지에 액세스할 때 큰 차이가 나기 때문이다.
3. 개체에 대한 액세스 제어 방법 선택 > Enforce public access prevent on this bucket 선택 취소 및 세분화 선택 후 생성
	*Enforce public access prevent on this bucket이 선택 취소된 경우에도 프로젝트의 조직 정책은 여전히 버킷 컨텐츠에 대한 공개 액세스를 거부할 수 있다.

#### 버킷에 이미지 파일 복사

1. Cloud Shell 활성화 및 이미지 복사
	- gsutil cp gs://cloud-training/gcpnet/cdn/cdn.png gs:// `bucket name`
2. 새로고침하여 이미지가 복사되었는지 확인
3. Cloud Storage 이미지 파일을 웹에 게시
	-  gsutil acl ch -u AllUsers:R gs://`bucket name`/cdn.png
4. 공개 액세스에서 공개 링크를 클릭하여 이미지에 액세스할 수 있는지 확인

### Cloud CDN으로 HTTP 부하 분산기 만들기

HTTP(S) 부하 분산은 Cloud Storage 버킷(백엔드)에 대한 정적 컨텐츠의 HTTP(S) 요청에 대한 전역 부하 분산을 제공한다.
백엔드에서 Cloud CDN을 사용 설정하면 컨텐츠가 일반적으로 백엔드보다 사용자에게 훨씬 더 가까운 Google Network Edge Location에 캐시된다.

#### HTTP 로드 밸런서 구성 시작

1. Google Cloud Console > Menu > Network Service > Load Balancers > CREATE LOAD BALANCER
2. HTTP(S) 부하 분산에서 구성 시작 및 이름 입력

#### 백엔드 구성

1. 백엔드 구성 > 백엔드 서비스 및 백엔드 버킷 > 백엔드 버킷 생성 및 이름 입력
2. Cloud Storage 버킷 > 찾아보기 > 전에 생성한 버킷 선택 > Cloud CDN 활성화 선택 후 생성

#### 프런트엔드 구성

호스트 및 경로 규칙은 트래픽이 전달되는 방식을 결정한다.
예를 들어 비디오 트래픽을 한 백엔드로 보내고 이미지 트래픽을 다른 백엔드로 보낼 수 있다.

1. 프런트엔드 구성
	1) Protocol: HTTP
	2) IP version: IPv4
	3) IP address Ephemeral
	4) Port: 80

#### HTTP 부하 분산기 검토 및 생성

1. 검토 및 완료 구성에서 백엔드 버킷 및 프런트엔드 검토 후 생성
2. 생성된 로드 밸런서를 클릭하여 로드 밸런서의 IP 주소 기록

### 버킷 컨텐츠 캐싱 확인

버킷용 HTTP 부하 분산기를 만들고 Cloud CDN을 사용 설정했으므로 이미지가 Google Network Edge에 캐시되었는지 확인해야 한다.

#### 이미지에 대한 HTTP 요청 시간

이미지가 캐시되었는지 확인하는 한 가지 방법은 이미지에 대한 HTTP 요청 시간을 확인하는 것이다.
컨텐츠는 해당 위치를 통해 액세스한 후 엣지 위치에서만 캐시되기 때문에 첫 번째 요청은 이후 요청에 비해 더 오래 걸린다.

1. Cloud Shell 활성화 및 로드 밸런서의 IP 주소를 환경 변수에 저장
	- export LB_IP_ADDRESS=`load balancer IP address`
2. 연속 HTTP 요청에 대해 명령 실행
	- for i in {1..3};do curl -s -w "%{time_total}\n" -o /dev/null http://$LB_IP_ADDRESS/cdn.png; done
	*첫 번째 출력이 두 번째 및 세 번째 출력 시간에 비해 오래 걸림을 알 수 있음

### Cloud CDN 로그

이전에 이미지가 캐시되었는지 확인하는 또 다른 방법은 Cloud CDN 로그를 탐색하는 것이다.
이러한 로그에는 컨텐츠가 캐시된 시기와 캐시에 액세스한 시기에 대한 정보가 포함된다.

1. Google Cloud Console > Logging > Log Explorer
2. 리소스 필터에서 Cloud HTTP Load Balancer > `load balancer forwarding rule` > `load balancer name`
3. 첫 번째 로그 항목(상단)을 확장하고 항목 내에서 httpRequest를 펼친 후 cacheLookup이 true이지만 cacheHit 필드가 없는지 확인
	*첫 번째 요청에는 cacheHit가 false이다. 이후 요청부터 CDN이 실행되어 cacheHit가 true로 변경된다(캐시에 첫 번째 요청의 이미지가 포함되지 않았음을 의미).
4. jsonPayload를 확장하고 statusDetails 필드에 response_sent_by_backend가 포함되어 있는지 확인
	*이미지가 첫 번째 요청의 백엔드 버킷에서 온 것임을 보여준다.
5. 현재 로그 항목을 닫고 다른 로그 항목을 확장한다.
	*이후 3번 및 4번 항목을 반복하여 확인
	캐시에 요청에 대한 이미지가 포함되어 있음을 나타내고, 백엔드 대신 캐시가 해당 요청에 이미지를 제공했음을 보여준다.

### 결론

HTTP 부하 분산기를 구성하고 간단한 체크박스로 Cloud CDN을 사용 설정하여 백엔드 버킷용 Cloud CDN을 구성하고, 이미지에 여러번 액세스하고 Cloud CDN 로그를 탐색하여 버킷 컨텐츠의 캐싱을 확인한다.
이미지에 처음 액세스할 때 엣지 로케이션의 캐시에 아직 이미지가 포함되지 않았기 때문에 시간이 오래 걸렸으며, 다른 모든 요청은 Cloud Shell 인스턴스에 가장 가까운 엣지 로케이션의 캐시에서 이미지가 제공되었기 때문에 빨랐음을 알 수 있다.
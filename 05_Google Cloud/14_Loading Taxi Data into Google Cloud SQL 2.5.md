
### 개요

CSV 텍스트 파일에서 Cloud SQL로 데이터를 가져온 다음 단순 쿼리를 사용하여 기본 데이터 분석을 수행하는 방법을 알아볼 것이다.
사용된 데이터 세트는 NYC Taxi & Limousine Commission에서 수집되었으며, 2009년부터 현재까지 뉴욕시의 모든 노란색 및 녹색 택시로 완료된 모든 운행 기록과 2015년부터 현재까지 임대 자동차(FHV)로 완료된 모든 운행 기록이 포함되어 있다.
승차 및 하차 날짜/시간, 승차 및 하차 위치, 이동 거리, 항목별 요금, 결제 유형, 운전자가 보고한 승객 인원을 수집하는 필드가 기록에 포함된다.

### 환경 준비

프로젝트 ID에서 향후 실습에 사용될 환경 변수와 데이터를 저장할 저장용량 버킷을 생성한다.
	```
	export PROJECT_ID=$(gcloud info --format='value(config.project)')
	export BUCKET=${PROJECT_ID}-ml
	```

### Cloud SQL 인스턴스 만들기

1. Cloud SQL 인스턴스 생성
	```
	gcloud sql instances create taxi \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS
	```
2. Cloud SQL 인스턴스의 루트 비밀번호 설정
	```
	gcloud sql users set-password root --host % --instance taxi \
	 --password Passw0rd
	```
*백슬래시 뒤에는 공백을 넣지 않도록 주의한다.
3. Cloud Shell의 IP 주소로 환경 변수 생성
	```
	export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32
	```
4. 관리를 위한 Cloud Shell 인스턴스를 허용 목록에 추가하여 SQL 인스턴스에 액세스한다.
	```
	gcloud sql instances patch taxi --authorized-networks $ADDRESS
	```
5. Cloud SQL 인스턴스의 IP 주소 가져오기
	```
	MYSQLIP=$(gcloud sql instances describe \
	taxi --format="value(ipAddresses.ipAddress)")
	```
6. MYSQLIP 변수 확인
	```
	echo $MYSQLIP
	```
7. MySQL 명령줄 인터페이스에 로그인하여 taxi trips 테이블 생성
	```
	mysql --host=$MYSQLIP --user=root \
      --password --verbose
	```
8. 비밀번호 입력 후 trips 테이블에 대한 스키마 생성
	```
	  create database if not exists bts;
	  use bts;
	  drop table if exists trips;
	  create table trips (
	    vendor_id VARCHAR(16),
	    pickup_datetime DATETIME,
	    dropoff_datetime DATETIME,
	    passenger_count INT,
	    trip_distance FLOAT,
	    rate_code VARCHAR(16),
	    store_and_fwd_flag VARCHAR(16),
	    payment_type VARCHAR(16),
	    fare_amount FLOAT,
	    extra FLOAT,
	    mta_tax FLOAT,
	    tip_amount FLOAT,
	    tolls_amount FLOAT,
	    imp_surcharge FLOAT,
	    total_amount FLOAT,
	    pickup_location_id VARCHAR(16),
	    dropoff_location_id VARCHAR(16)
	  );
	```
9. MySQL 명령줄 인터페이스에서 가져온 항목 확인
	```
	describe trips;
	```
10. trips 테이블 쿼리
	```
	select
	distinct(pickup_location_id)
	from trips;
	```

### Cloud SQL 인스턴스에 데이터 추가하기

1. 택시 운행 CSV 파일을 Cloud Storage에 로컬로 저장하고, 리소스 사용을 줄이기 위해 데이터의 하위 세트(2만 행 이하)로만 작업한다.
	```
	gsutil cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv trips.csv-1
	gsutil cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv trips.csv-2
	```
2. MySQL을 사용하여 Cloud SQL로 CSV 파일 데이터 가져오기
	```
	mysql --host=$MYSQLIP --user=root  --password  --local-infile
	```
3. MySQL 대화형 콘솔에 연결
	```
	use bts;
	
	LOAD DATA LOCAL INFILE 'trips.csv-1' INTO TABLE trips
	FIELDS TERMINATED BY ','
	LINES TERMINATED BY '\n'
	IGNORE 1 LINES
	(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
	
	LOAD DATA LOCAL INFILE 'trips.csv-2' INTO TABLE trips
	FIELDS TERMINATED BY ','
	LINES TERMINATED BY '\n'
	IGNORE 1 LINES
	(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);
	```

### 데이터 무결성 확인하기

데이터를 소스에서 가져올 때마다 데이터 무결성을 확인하는 것은 항상 중요하다.
이는 대략적으로 데이터가 기대에 부합하게 만드는 것이다.

1. 고유한 픽업 위치 영역에 대해 trips 테이블을 쿼리한다.
	```
	select distinct(pickup_location_id) from trips;
	```
2. trip_distance 열에서 택시의 최대 및 최소 운행 거리 살펴보기
	```
	select
	  max(trip_distance),
	  min(trip_distance)
	from
	  trips;
	```
3. 데이터베이스에 운행 거리가 0인 운행 살펴보기
	```
	select count(*) from trips where trip_distance = 0;
	```
4. 데이터베이스에 운행 거리가 0 이하로 반환되는 값 살펴보기
	```
	select count(*) from trips where fare_amount < 0;
	```
5. 결제 유형 살펴보기
	```
	select
	  payment_type,
	  count(*)
	from
	  trips
	group by
	  payment_type;
	```
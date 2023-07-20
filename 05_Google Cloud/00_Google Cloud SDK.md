
### Google Cloud SDK Install

1. Google Cloud SDK 홈페이지에서 x86_64 다운로드
2. 터미널에서 python -V를 통해 파이썬 버전 확인
3. echo $SHELL을 통해 bash인지 zsh인지 확인
4. (zsh 일때) vi ~/.zshrc 입력 후 아래 코드 입력
	*bash를 사용하고 있는 경우 vi ~/.bashrc
	```
	## Google Cloud
	export GOOGLE_CLOUD_SDK_PATH=[google-cloud-sdk를 넣어둔 경로]
	export PATH=$PATH:$GOOGLE_CLOUD_SDK_PATH/bin
	```
	*tar 파일이 있는 폴더의 경로
	example) /Users/latency/desktop/Latency/google-cloud-sdk
5. 변경된 사항을 적용하기 위해 source ~/.zshrc 명령어 사용
	*bash를 사용하고 있는 경우 source ~/.bashrc
6. gcloud --version을 입력하여 설치가 완료되었는지 확인
7. gcloud 초기화
	본인의 계정과 프로젝트를 연결
	- gcloud init
8. Pick configuration to use & Google Login 설정
9. 로그인이 되었으면 본인이 연결하고자 하는 프로젝트 선택
	*Compute Engine 설정을 할건지에 대한 물음
	--> region/zone 설정을 default로 할 경우 n 입력

### Cloud에 파일 업로드

*아래 코드는 모두 예시입니다.
1. 인스턴스 생성
	- gcloud compute instances create `instance name` --machine-type `machine type` --zone `zone name`
	example) gcloud compute instances create test-instance --machine-type e2-medium --zone us-central1-a
2. txt 파일 생성
	example) echo "hello" > hello.txt
3. 현재 디렉터리에 생성한 파일이 있는지 확인
	- ls
4. 버킷 생성
	- gsutil mb gs://`bucket name`
	example) gsutil mb gs://my-bucket-00000000
5. 생성한 버킷으로 txt 파일 복사
	example) gsutil cp hello.txt gs://my-bucket-00000000
6. 파일이 복사 되었는지 디렉터리 확인
	example) gsutil ls gs://my-bucket-00000000
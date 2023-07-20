*CLI: Command Line Interface
*SDK: Software Development Kit

Google Cloud Storage를 사용하면 데이터 양에 상관없이 언제 어디서나 데이터를 저장하고 가져올 수 있다.
Google Cloud Storage를 웹사이트 콘텐츠 제공, 보관 및 재해 복구를 위한 데이터 저장, 직접 다운로드를 통한 사용자에게 대량의 데이터 객체 배포 등 다양한 용도로 사용할 수 있다.

### 버킷에 객체 업로드하기

1. 버킷을 만든 후 객체를 업로드 한다.
	*이미지를 Cloud Shell의 임시 인스턴트로 다운로드
	- wget --output-document `file name`.jpg `website link`
	example) wget --output-document ada.jpg https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg
2. gsutil cp 명령어를 사용하여 이미지를 현재 저장된 위치에서 버킷으로 업로드
	- gsutil cp ada.jpg gs://`bucket name`
	*명령줄에서 버킷에 대한 이미지 로드를 볼 수 있고, 객체가 버킷에 저장됨
3. 다운로드한 이미지 제거
	rm `file name`.jpg

### 버킷에서 객체 다운로드하기

1. gsutil cp 명령어를 사용하여 버킷에 저장한 이미지를 Cloud Shell로 다운로드
	- gsutil cp -r gs://`bucket name`/`file name`.jpg

### 버킷의 폴더에 객체 복사하기

1. gsutil cp 명령어를 사용하여 image-folder라는 폴더를 만들고 이미지 파일을 복사
	- gsutil cp gs://`bucket name`/`file name`.jpg gs://`bucket name`/image-foler/

### 버킷 또는 폴더 컨텐츠 목록 표시하기

1. gsutil 명령을 사용하여 버킷의 내용 표시
	- gsutil ls gs://`bucket name`

### 객체 세부정보 표시하기

1. 버킷에 업로드한 이미지 파일에 대한 세부정보를 얻으려면 -l 플래그와 함께 gsutil ls 명령을 사용한다.
	- gsutil ls -l gs://`bucket name`/`file name`.jpg
	*이미지의 크기와 생성 날짜를 알 수 있음

### 객체에 공개적인 액세스가 가능하도록 설정하기

1. gsutil acl ch 명령어를 사용하여 모든 사용자에게 버킷에 저장된 객체에 대한 읽기 권한 부여
	- gsutil acl ch -u AllUsers:R gs://`bucket name`/`file name`.jpg
	*ACL Reference Link: https://cloud.google.com/storage/docs/gsutil/commands/acl
	*<:R --> READ access> 이미지가 공개되어 누구나 사용할 수 있게 됨

### 공개 액세스 권한 삭제하기

1. 부여된 권한 삭제
	- gsutil acl ch -d AllUsers gs://`bucket name`/`file name`.jpg
	*해당 객체로의 공개 액세스 권한을 삭제하였고, Console에서 새로고침 버튼을 클릭하여 확인할 수 있음
	*체크박스가 선택 해제되었고, 이미지를 표시한 페이지를 새로고침하면 오류가 표시된다.

### 객체 삭제하기

1. gsutil rm 명령을 사용하여 버킷의 이미지 파일인 객체 삭제
	- gsutil rm gs://`bucket name`/`file name`.jpg
	*Console을 새로고침하면 더이상 이미지 파일의 복사본이 Cloud Storage에 저장되어 있지 않게 되지만, image-folder 폴더에 만든 사본은 계속 남아있다.


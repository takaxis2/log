아파치의 기능중 하나인 virtual host를 사용하면 도메인에 따라 여러개의 웹사이트를 운영하기 편하다. 그런데 막상 도메인에 대한 폴더를 만들고 테스트를 해보면 403 오류가 나오는 경우가 있다

## 사례

	1. .conf 파일
	2. 리눅스 권한


###  .conf 파일
```
아파치 virtual host 설정 파일을 확인한다. 
이때 설치된 아파치의 버전이나 사용자의 세팅에 따라 이름이나 경로가 다를 수 있다.
(vhosts.conf 또는 httpd.conf)
<Directory> 태그 안에 Require이 있는 문장을 확인한다.

```
액세스 제한은 Require로 작성을 한다. all를 사용하여 모두 허용(granted) 하거나 모두 거부(denied)할 수 있다. 그리고 host, ip 를 사용하여 특정 호스트나 IP 주소에 대해 허용 및 거부할 수 있다.

|형식|의미|
|---|---|
|Require all granted|모든 액세스 허용|
|Require all denied|모든 액세스 거부|
|Require ip IP-주소|해당 IP 주소 허용|
|Require not ip IP-주소|해당 IP 주소 거부|
|Require host 호스트|해당 호스트 허용|
|Require not host 호스트|해당 호스트 거부|
### 리눅스 권한
```
예상치 못한 문제라 원인ㅇ
```
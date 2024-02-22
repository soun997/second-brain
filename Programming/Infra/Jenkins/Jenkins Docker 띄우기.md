
> Ubuntu Jammy 22.04 (LTS)

**Docker가 설치되어있음을 전제로 합니다.**

### Docker Hub에서 Jenkins 이미지 pull

```bash
sudo docker pull jenkins/jenkins:jdk17
```

서버의 경우 JDK17 버전을 사용할 것이기 때문에 jdk17 태그를 달아주었다.

이미지를 hub에서 내려받게 된다.

### Jenkins Docker Container 띄우기

``` bash
sudo docker run -v /var/run/docker.sock:/var/run/docker.sock -d -p 8090:8080 -p 50000:50000 --name jenkins --restart=on-failure soun997/jenkins-docker
```

서버 포트에 이미 8080 포트를 할당했기 때문에, Host 포트를 8090으로 바꾸어주었다. - [[Docker Port]] 참고

### Jenkins initial password 알아내기

``` bash
sudo docker logs {CONTAINER_ID}
```

실행 중인 Docker 컨테이너의 로그를 출력한다.

최초로 실행했을 경우 initial password를 알려주기 때문에 
이를 복사해서 Jenkins Dashboard에 로그인하면 된다.


### Jenkins 로그인 및 Plugin 설치

복사한 initial password를 통해 로그인하면, 플러그인 설치 페이지로 넘어가게 된다.

원하는 플러그인 만을 선택해서 설치할 수도 있지만, 일단은 추천해주는 플러그인을 모두 설치하자!


## [[Docker out of Docker (DooD)]] 방식을 적용하여 Jenkins 구동

1. docker_install.sh 작성
2. Dockerfile 작성
3. 이미지 생성
4. 이미지 Push
5. 컨테이너 실행
	1. docker.sock 연결
6. 컨테이너 접속하여 docker.sock 권한 부여``
	`docker exec -itu 0 <컨테이너 ID> sh -c 'setfacl -Rm d:g:docker:rwx,g:docker:rwx /var/run/'`
1. 컨테이너 접속하여 docker compose 설치
	1. https://docs.docker.com/compose/install/


## Jenkins Nginx 설정

``` bash
upstream jenkins {
	keepalive 32; # keepalive connections
	server 127.0.0.1:8090; # Jenkins의 IP와 Port
}
# Required for Jenkins websocket agents
map $http_upgrade $connection_upgrade {
	default upgrade;
	'' close;
}
server {
	listen 80; # Listen on port 80 for IPv4 requests
	server_name jenkins.api.zupzup.shop; # 자기 서버 주소로 변경

	# this is the jenkins web root directory
	# (mentioned in the output of "systemctl cat jenkins")
	root /var/run/jenkins/war/;
	access_log /var/log/nginx/jenkins.access.log;
	error_log /var/log/nginx/jenkins.error.log;
	# pass through headers from Jenkins that Nginx considers invalid
	ignore_invalid_headers off;

	location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {
		# rewrite all static files into requests to the root
		# E.g /static/12345678/css/something.css will become /css/something.css
		rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;
	}

	location /userContent {
		# have nginx handle all the static requests to userContent folder
		# note : This is the $JENKINS_HOME dir
		root /var/lib/jenkins/;

		if (!-f $request_filename){
			# this file does not exist, might be a directory or a /**view** url
			rewrite (.*) /$1 last;
			break;
		}

		sendfile on;
	}

	location / {
		sendfile off;
		proxy_pass http://jenkins;
		proxy_redirect default;
		proxy_http_version 1.1;

		# Required for Jenkins websocket agents
		proxy_set_header Connection $connection_upgrade;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_max_temp_file_size 0;

		#this is the maximum upload size
		client_max_body_size 10m;
		client_body_buffer_size 128k;
		proxy_connect_timeout 90;
		proxy_send_timeout 90;
		proxy_read_timeout 90;
		proxy_buffering off;
		proxy_request_buffering off; # Required for HTTP CLI commands
		proxy_set_header Connection ""; # Clear for keepalive
	}
}
```


## Ref

[Dockerhub-jenkins](https://hub.docker.com/r/jenkins/jenkins/tags)
[Jenkins Documentation](https://github.com/jenkinsci/docker/blob/master/README.md)
[Docker Out of Docker (DooD)](https://www.skyer9.pe.kr/wordpress/?p=3382)
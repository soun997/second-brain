
``` bash
sudo docker run -p 80:5000 centos:latest
```

## Host Port

위의 커맨드에서 `80`포트를 의미한다.

Docker를 설치한 호스트의 포트이다.


## Container Port

위의 커맨드에서 `5000`포트를 의미한다.

Docker 컨테이너의 포트이다.


## Host : Container의 의미

Docker를 설치한 호스트의 포트를 컨테이너에 연결하겠다는 뜻이다.

만약 Jenkins Docker를 실행할 때, `-p` 옵션을 `8090:8080` 과 같이 설정했다면
호스트에서 `8090` 포트를 사용해 Jenkins Dashboard에 접근할 수 있다.

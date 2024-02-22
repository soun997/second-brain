
Docker는 크게 Docker Host, Docker Client, Docker Registry로 분리되어 있다.

![[Pasted image 20231127053022.png]]


## Docker Host (dockerd)

Docker Daemon이라고 부르기도 한다.

**Docker API 요청을 수신**하고
이미지, 컨테이너, 네트워크, 볼륨 등의 도커 오브젝트를 관리한다.


## Docker Client (docker)

**Docker 서버와 통신**하기 위한 기능을 수행한다.

docker 명령어를 수행 할 경우
Docker Client가 해당 명령어를 REST API Call로 변환하여 **Docker Host로 전송**한다.


## Docker Registry

**Docker 이미지를 저장하는 저장소 역할을 한다.**

default로 Public Docker Registry인 Docker Hub를 사용한다.
- 사용자가 Private Registry를 구축하여 사용할 수 있다.

`docker pull`, `docker run`과 같은 명령어를 사용하면 
사용자가 요청한 이미지를 Docker Registry에서 찾아서 가져온다.

`docker push` 명령어를 사용하면 docker는 이미지를 Docker Registry에 저장한다.


## Ref
[컨테이너 내부에서 Docker 사용하기 - DinD, DooD](https://velog.io/@inhwa1025/Docker-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EB%82%B4%EB%B6%80%EC%97%90%EC%84%9C-Docker-%EC%82%AC%EC%9A%A9)

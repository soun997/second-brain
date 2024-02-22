
사용 중인 Docker의 기능을 빌려서 사용하는 Docker out of Docker
- 즉, 컨테이너가 자신이 속한 Docker의 서비스를 사용할 수 있다.
- [[Docker in Docker (DinD)]] 의 보안 이슈로 인해 DooD를 권장한다고 한다. (보안상 허점은 여전히 존재)

Docker의 `docker.sock` 파일을 컨테이너와 공유함으로써 도커 명령을 사용할 수 있다.

``` bash
docker run -d -p 9090:8080 --name=jenkinscicd \ 
-v ~/jenkins_home_sub:/var/jenkins_home \ 
-v /var/run/docker.sock:/var/run/docker.sock \ 
jenkins/test_jenkins
```


## Ref
[Using Docker-in-Docker for your CI or testing environment? Think twice.](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
[docker in docker 와 docker out of docker](https://rainbound.tistory.com/entry/docker-in-docker)
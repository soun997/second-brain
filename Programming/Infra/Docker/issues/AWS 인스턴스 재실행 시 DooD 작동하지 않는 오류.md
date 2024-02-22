## 문제 상황

``` sh
+ docker compose -f docker-compose.yml down --rmi all
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "[http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1&filters=%7B%22label%22%3A%7B%22com.docker.compose.config-hash%22%3Atrue%2C%22com.docker.compose.oneoff%3DFalse%22%3Atrue%2C%22com.docker.compose.project%3Dveganlife%22%3Atrue%7D%7D](http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json?all=1&filters=%7B%22label%22%3A%7B%22com.docker.compose.config-hash%22%3Atrue%2C%22com.docker.compose.oneoff%3DFalse%22%3Atrue%2C%22com.docker.compose.project%3Dveganlife%22%3Atrue%7D%7D)": dial unix /var/run/docker.sock: connect: permission denied
```

외부의 `docker.sock` 파일을 사용할 권한이 없기 때문에 발생하는 오류이다.
(외부의 `docker.sock`과 연결은 가능한 상태이다!!)


## 해결 방법

[Docker Out of Docker (DooD)](https://www.skyer9.pe.kr/wordpress/?p=3382)
(내가 처음 DooD를 구현하기 위해 참고했던 블로그이다.)

`docker.sock`에 대한 권한을 부여해주기만 하면 된다!

```bash
docker exec -itu 0 jenkins sh -c 'setfacl -Rm d:g:docker:rwx,g:docker:rwx /var/run/'
```
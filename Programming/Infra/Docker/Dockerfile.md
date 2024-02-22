
>[[Docker Image]]를 빌드하는 방법에 대한 스크립트


## 환경변수

Docker Container는 이미지의 실행 가능한 인스턴스이다.
- 필요에 따라 Bulid 시간 혹은 Runtime 환경변수가 필요하다.

Dockerfile에 환경변수를 주입하는 방식의 장점은 다음과 같다.
1. 유연성
	1. 환경(dev 혹은 prod 등)에 따른 Dockerfile을 만들어 사용할 수 있다.
2. 보안
	1. 민감한 정보를 Dockerfile에 직접 저장하지 않아도 된다.

### 환경변수 주입하기

Dockerfile은 환경변수를 위한 전용 변수인 `ENV`변수를 제공한다.
- Build time, Runtime 모두 `ENV`변수에 **액세스 가능**하다.

#### Bulid 시 환경변수가 주입되지 않는 문제

Jenkins Pipeline 실행 도중, build task의 test 실행에 계속 실패했다.
로그를 분석한 결과 환경변수가 제대로 주입되지 않음을 확인했다.
(오류에 대한 자세한 내용은 이곳을 참고하길 바란다 [[Jenkins - Trouble Shooting]])

왜 그랬을까?

**Dockerfile은 빌드 프로세스 중에 `ENV`변수를 동적으로 설정할 수 없다.**
- `ENV MARIA_URL ${MARIA_URL}` 이런 식으로 둬도 못 읽어온다.
- 왜냐하면 `ENV` 변수는 실행 시점만 주입할 수 있는 친구이기 때문이다.
- Build 시점에 `ENV` 변수를 사용하려면 하드 코딩하는 방법 밖엔 없다.

이를 해결하기 위해 `ARG` 변수를 사용할 수 있다.
- Build 시점에만 주입할 수 있는 환경변수이다.
- `ARG` 변수는 이미지가 빌드되면 더 이상 액세스 불가하다. (Build 시점에만 사용 가능)

```Dockerfile
ARG MARIA_URL
ARG MARIA_USER
ARG MARIA_PASSWORD
```

`ARG` 변수에 값을 주입하는 방법은 두 가지가 있다.

1. `docker build` 명령어를 사용할 때 `--build-arg` 옵션을 이용해 `ARG` 변수에 값을 넘겨줄 수 있다.

``` bash
docker build --build-arg MARIA_URL=$(MARIA_URL)
```

2. 혹은 `docker-compose.yml`에서 `args:`에 환경변수를 정의하여 `ARG` 변수에 값을 넘겨줄 수 있다. 

```yaml
version: "3"
services:
	spring:
		...
		build:
			args:
				MARIA_URL: ${MARIA_URL}
				MARIA_USER: ${MARIA_USER}
				MARIA_PASSWORD: ${MARIA_PASSWORD}
```

### `ENV` VS `ARG`

둘의 차이를 정리하면 다음과 같다.

#### `ENV`
- Build, Runtime 시점에 사용되는 변수 (단, Build 시점에는 동적 바인딩 불가)
- `$변수` 혹은 `${변수}` 형태로 표현
- `docker run` 시에 `--e` 옵션을 활용하여 오버라이딩(동적바인딩) 가능

#### `ARG`
- Build 시점에만 사용되는 변수
- `ARG 변수` 혹은 `ARG 변수=값` 형태로 표현 가능
- `docker build` 시에 `--build-arg` 옵션을 활용하여 오버라이딩(동적바인딩) 가능

참고로 `ENV`와 `ARG` 둘 다 프로그램의 명령줄 인수로 들어가는 것은 같다.

### Ref

[How to Pass Environment Variable Value into Dockerfile](https://www.baeldung.com/ops/dockerfile-env-variable)
- 번역) [환경 변수 값을 Dockerfile에 전달하는 방법](https://recordsoflife.tistory.com/755)
[ARG](https://docs.docker.com/engine/reference/builder/#arg)
[ENV](https://docs.docker.com/engine/reference/builder/#env)
[How to use Docker Build Args and Environment Variables](https://refine.dev/blog/docker-build-args-and-env-vars/#using-env-file)
- 맛도리
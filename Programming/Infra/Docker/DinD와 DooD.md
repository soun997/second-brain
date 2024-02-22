
Docker 관련 명령어를 수행하기 위해서는 Docker [[Daemon]]의 도움이 필요하다. (참고: [[Docker 아키텍처]])

만약 Docker 컨테이너 안에서 Docker 관련 명령어를 수행해야 한다면 어떻게 해야할까?
(실제로 CI를 위해 컨테이너 내부에서 Docker 명령어를 실행해야 하는 경우가 꽤 있다. ex. Jenkins)

이를 위해 아래의 2가지의 방법 중 하나를 택할 수 있다.
1. [[Docker in Docker (DinD)]]
2. [[Docker out of Docker (DooD)]] (Recommended)



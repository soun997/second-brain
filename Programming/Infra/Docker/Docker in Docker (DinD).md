
Docker 컨테이너 내에서 Docker engine을 실행하는 방식이다.

즉, 로컬 호스트에서 이용하는 Docker와 다른 별도의 Docker를 컨테이너 내부에 설치하여 사용하는 것이다.

Docker 컨테이너는 기본적으로 Unprivileged 모드로 실행되기 때문에, Host의 시스템 자원에 접근할 수 없다.

## Unprivileged 상태에서 [DinD 이미지](https://hub.docker.com/_/docker)를 띄웠을 때

실행하고 얼마 안 있어 컨테이너가 종료된다.

로그를 찍어보면,

```
Certificate request self-signature ok
subject=CN = docker:dind server
/certs/server/cert.pem: OK
Certificate request self-signature ok
subject=CN = docker:dind client
/certs/client/cert.pem: OK
ip: can't find device 'ip_tables'
ip_tables              36864 10 iptable_filter,iptable_nat
x_tables               65536 16 iptable_filter,iptable_nat,xt_nat,xt_MASQUERADE,ip6t_REJECT,xt_hl,ip6_tables,ip6t_rt,ipt_REJECT,xt_LOG,xt_limit,xt_addrtype,xt_tcpudp,xt_conntrack,nft_compat,ip_tables
modprobe: can't change directory to '/lib/modules': No such file or directory
mount: permission denied (are you root?)
Could not mount /sys/kernel/security.
AppArmor detection and --privileged mode might break.
mount: permission denied (are you root?)
```

`ip_tables`이라는 device을 찾을 수 없다고 나온다.

### 나의 추측

Docker Daemon은 Docker Client의 API 요청을 수신하는 역할을 한다.
그렇기 때문에 Network 정보를 필요로 할 것이다.

그러나 Network 정보를 알기 위해 ip_tables에 접근했지만, 
기기 정보를 알 수 있는 권한이 없기에 터진게 아닐까 싶다.


그렇다면 어떻게 해야 Docker Daemon을 사용할 수 있을까?

## `--previleged`



## Ref
[Docker in Docker and play-with-docker](https://sreeninet.wordpress.com/2016/12/23/docker-in-docker-and-play-with-docker/)
[Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)

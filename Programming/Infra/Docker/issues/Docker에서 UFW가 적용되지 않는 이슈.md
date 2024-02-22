
Docker에서 [[UFW]]가 적용되지 않는 문제가 발견되었다.


## 발생 이유

#### 도커는 어떻게 UFW 방화벽을 우회하여 포트를 개방할 수 있었나?

![[Pasted image 20231206224930.png|]]

[[IPTABLES]]의 Filter Table에 생성된 Chain을 확인해보자.
위의 그림에서 UFW와 Docker 관련 Chain이 격리되어 있음을 확인할 수 있다.

다음 명령을 통해 IPTABLES의 규칙을 알아보자

``` bash
sudo iptables -L
```


![[Pasted image 20231206225357.png]]

각각의 체인이 무엇을 의미하는지 간략하게 살펴보자.

**INPUT Chain**
- 외부에서 방화벽 서버로 들어오며 거치는 패킷

**FORWARD Chain**
- 서버가 목적이 아닌 다른 네트워크로 포워딩 목적으로 방화벽을 통해 흘러가며 거치는 패킷

**OUTPUT Chain**
- 서버에서 외부로 나가며 거치는 패킷

**IPTABLES는 등록된 순서가 중요하다.**
- FORWARD Chain을 보면, DOCKER-USER가 1순위로 등록되어 있음을 알 수 있다.
- UFW 이전에 등록되어 있기 때문에, UFW의 정책에 구애 받지 않는다.

![[Pasted image 20231207012107.png|500]]

## 어떻게 해결할 수 있나?

### `"iptables" : false`로 설정

IPTABLES과의 관계를 직접적으로 끊는다.

![[Pasted image 20231207012119.png|500]]

단, IPTABLES을 끊음으로서 라우터 기능이 손실되게 된다.

``` bash
sudo ufw status verbose
```

![[Pasted image 20231207012517.png]]

UFW의 routed를 허용하면 route 기능을 활성화 할 수 있다.

``` bash
sudo ufw default allow routed
sudo ufw reload
```

그러나 도커 서비스를 재실행 할 경우 설정이 꼬이는 문제가 발생하는 듯하다.


### [[ufw-docker]]를 사용 (권장)

UFW 설정을 통해 DOCKER-USER Chain의 설정을 변경한다.
- 설정이 꽤 복잡하지만, ufw-docker라는 util을 이용하여 간단하게 처리할 수 있다.

다음과 같은 과정을 거친다.
1. 도커로 접근을 하면
2. ufw-user-forward 에서 1차로 IP 또는 Port번호를 식별해서 **Allow, Deny 조치**합니다.
3. 사설 네트워크 대역에서 접근한 것이라면 ? Ok! 들어와! **하이패스** 시켜줍니다.
4. 위에서 부터 1차로 **FORWARD 거르고**, 2차로 **사설 네트워크인** 것을 걸렀는데 여기까지 왔네? 도커 컨테이너를 이용하겠다고? 안돼! **입장불가! DROP**


## Ref

[Docker + UFW 적용 문제와 해결 방법](https://d-life93.tistory.com/431)
[Docker + UFW+ IPTABLES 방화벽 적용](https://d-life93.tistory.com/466)


`default_server`는 단 하나만 정의할 수 있다.

그러나 되도록이면 `default_server`로 접속할 경우 에러 페이지로 이동하도록 하는 것이 좋다.

> 이렇게 default_server가 막혀있지 않은 서버의 경우에는 서버에 등록되지 않은 호스트로 접근되었을 때 어떻게 fallback되어 무슨 일을 일으킬 수 있을지 모릅니다.

나는 HTTP 프로토콜(80포트)로 접근하는 모든 연결들을 불완전한 연결로 보고
아래처럼 `default_server`를 이용하여 에러페이지로 연결되도록 했다.

``` bash
server {

        listen 80 default_server;
        listen [::]:80 default_server;

        # Everything is a 444
        location / {
                return 444;
        }

}
```


## 무엇이 문제가 되는가?

가비아에 등록된 도메인은 `dev.konggogi.store`, `prod.konggogi.store` 두 개다.
현재 설정 파일을 SSL 인증을 받은 서버는 `dev.konggogi.store` 뿐이다.

위의 `default_server` 설정을 통해 HTTP 프로토콜로 들어오는 접근들은 모두 에러 페이지로 이동하게 된다.

**제대로 동작하는 경우**
`http://prod.konggogi.store`로 접속 시, 에러 페이지로 제대로 이동함을 확인했다.

![[Pasted image 20231208001537.png]]


**제대로 동작하지 않는 경우**
`http://prod.konggogi.store`로 접속 시, 안전하지 않은 상태로 Greeting 페이지에 연결되었다.

![[Pasted image 20231208001713.png]]

인증은 성공하지 못했지만, 실패했다고 따로 에러 페이지로 보내는 로직이 없었기 때문에, index 페이지로 넘어가게 되었다.

![[Pasted image 20231208002358.png]]

그러므로, 각 Nginx 설정 파일에 아래 항목을 추가해주는 것이 불완전한 연결을 예방할 수 있다.
- 여기서 말하는 예방이란, 에러 페이지로 이동하도록 하는 것이다.

``` bash
if ($host != $server_name) {
        return 444;
}
```


## Ref

[nginx에서 default_server를 막아야 하는 이유 (+ https)](http://linforum.kr/bbs/board.php?bo_table=security&wr_id=104)
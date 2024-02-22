
## 문제

[[Jenkins Pipeline]]에서는 Health Check가 잘 작동했다.
그런데 브라우저에 URL을 입력해서 Health Check 페이지를 접속하려고 하니 접속이 안됐다!
- Empty Response Error가 발생한다.

## 발생 이유

Empty Response는 api와 관련 없는 경로로 접속을 시도할 때 발생하는 에러이다.
- 내가 Nginx에 설정해두었다.

Nginx 설정을 확인해보니 역시 `/api` 형식으로 들어오는 URL만 backend 서버로 연결되도록 설정해두었다.

Jenkins에서는 도메인명이 아닌 `localhost:8080`으로 서버에 접속했기 때문에 Nginx 프록시 서버를 거치지 않아 Pipeline의 진행에 문제가 없었다.

## 해결?

해결할 필요가 없다.
왜냐하면 Health Check 정보를 사용자들에게 노출할 필요가 없기 때문이다.

굳이 외부에서 접근하게 하지 말고, 로컬 내의 서비스들끼리 호출하도록 해야겠다.
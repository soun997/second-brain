## `npm`을 사용할 수 없다.

`npm`의 `jsonwebtoken` 라이브러리를 사용하려고 했지만 쓸 수 없었다.

## JWT 토큰 직접 생성

직접 `header`, `payload`, `secret` 기반으로 토큰을 생성하는 로직을 짜주어야 한다.
아래를 참고하여 작성했다.
https://gist.github.com/robingustafsson/7dd6463d85efdddbb0e4bcd3ecc706e1

## 추가

보니까 OAuth 같은 것도 처리할 수 있도록 해둔 것 같다.
https://k6.io/docs/examples/oauth-authentication/

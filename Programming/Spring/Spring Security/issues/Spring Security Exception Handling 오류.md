## 문제 상황

인증이 필요한 request uri이지만 인증에 성공하지 못했을 경우의 Excepiton Handling이 제대로 되지 않는다.
- 연관이 없어보이는 여러 예외가 중첩되어 발생했다.

**핸들링 되지 않은 예외**
- `AccessDeniedException` - `AuthroizationFilter`에서 발생
- `NullPointerException` - `JwtAuthenticationEntryPoint`의 `commence` 를 수행하는 도중 ErrorCode가 null이기 때문에 발생

**Security 필터 체인**
```
Security filter chain: [
  DisableEncodeUrlFilter
  WebAsyncManagerIntegrationFilter
  SecurityContextHolderFilter
  HeaderWriterFilter
  CorsFilter
  LogoutFilter
  JwtAuthenticationFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  SessionManagementFilter
  ExceptionTranslationFilter
  AuthorizationFilter
]
```

## 발생 이유

### 인증 예외 처리 과정
1. `JwtAuthenticationFilter`에서 인증 처리를 시도한다.
	1. ==Authorization 헤더가 없다면 예외처리가 되지 않는다.== (null or Bearer 붙어있지 않음) - 해결함
	2. Authorization 헤더가 유효하지 않다면 request에 ErrorCode를 저장하고 필터를 진행시킨다.
		1. ==계속 필터를 타고 내려간다.==
		2. `AuthorizationFilter`에 도착, security context가 존재하지 않으므로 `AccessDeniedException` 발생
		3. 인증되지 않은 사용자는 곧 Anonymous 사용자이기 때문에 `ExceptionTranslationFilter` 필터에서 security context를 생성하고, `JwtAuthenticationEntryPoint`의 `commence`를 호출한다.
		4. `JwtAuthenticationEntryPoint`의 `commence`에서 request 내의 `ErrorCode`를 바탕으로 error response를 commit한다.
		5. 필터 단에서 예외가 발생하였으므로 Dispatcher Servlet으로 넘어가지 않고 필터를 타고 올라간다.
			1. 최종적으로 error를 담은 response를 응답한다.

디버깅 로그를 찍어본 결과, Authorization 헤더는 SSE 재연결 요청에도 제대로 들어가는 것을 확인했다
그렇다면 security context 또한 존재해야 하는데 왜 `JwtAuthenticationEntryPoint`의 `commence`가 실행되는 것일까?

### SSE 재연결 요청일 경우

`JwtAuthenticationFilter`를 거치지 않음을 확인했다.
때문에 Anonymous 사용자로 인식이 되고, 위의 인증 예외 처리 과정을 거친다.

그러나 `JwtAuthenticationEntryPoint`의 `commence`는 `ErrorCode`가 있는 상황에만 정상 작동한다. `JwtAuthenticationFilter`를 거치지 않아 request에 `ErrorCode`가 없기 때문에 `commence`를 실행하는 과정에서 `NullPointerException`이 발생한다.

결국 `/error`로 에러 요청을 보내지만 해당 리소스는 존재하지 않으로 또 예외가 발생하였기 때문에 중첩해서 예외가 발생한 것이였다.


## 해결 방법

### `JwtAuthenticationFilter`를 거치도록

### 왜 필터를 거치지 않는가?

재연결 요청은 비동기적으로 수행되기 때문이다. (언제 Timeout 되어 재연결 요청이 올 지 모르므로)

```java
// OncePerRequestFilter.java

@Override  
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)  
       throws ServletException, IOException {  
	...
    if (skipDispatch(httpRequest) || shouldNotFilter(httpRequest)) {  
       // Proceed without invoking this filter...  
       filterChain.doFilter(request, response);  
    }
    ...
```

`OncePerRequestFilter` 클래스의 `doFilter` 메서드에는 `skipDispatch`와 `shouldNotFilter` 메서드가 존재한다.
둘 중 하나라도 충족하게 되면 해당 필터를 무시한다.

```java
private boolean skipDispatch(HttpServletRequest request) {  
    if (isAsyncDispatch(request) && shouldNotFilterAsyncDispatch()) {  
       return true;  
    }  
    if (request.getAttribute(WebUtils.ERROR_REQUEST_URI_ATTRIBUTE) != null && shouldNotFilterErrorDispatch()) {  
       return true;  
    }  
    return false;  
}
```

`skipDispatch` 메서드를 확인해보면 ==요청이 비동기==이고, ==비동기 요청은 필터링하지 않겠다는 옵션이 켜져있으면== 해당 필터를 거치지 않는다.

### 해결

```java
// JwtAuthenticationFilter.java
...
@Override
protected boolean shouldNotFilterAsyncDispatch() {  
    return false;  
}
...
```

비동기 요청 또한 필터링하도록 `protected` 메서드를 오버라이딩 해주었다.  (단, 이로 인한 부작용이 무엇일지는 아직 모른다)


## 그래도 예외는 계속된다...

### AsyncRequestTimeoutException 핸들링

SSE 연결이 timeout되면 클라이언트는 재연결을 서버에게 요청한다.
필터 단은 무사히 통과하지만, Dispatcher Servlet으로 넘어가면서 `AsyncRequestTimeoutException`이 발생한다. (Controller에 넘어가기도 전이다, 어떤 로직으로 발생하는지는 아직 모르겠다.)

### 해결

Dispatcher Servlet에서 발생하기 떄문에 `@@ExceptionHandler(AsyncRequestTimeoutException.class)`로 처리해주었다.


**대략적인 예외 핸들링은 처리 완료!**


## 추가적인 이슈

1. `AccessDeniedHandler`가 구현되어 있지 않다.
    1. 인가 예외 처리를 위해서는 구현해야 할 것 같다.
2.  필터 단에서 발생한 예외나 `ExceptionHandler`로 핸들링되지 않은 Servlet 단 예외는 무조건 `/error`로 에러 요청을 보낸다, 없으니까 무조건 403 error)
3. `JwtAuthenticationEntrypoint`에서만 예외를 처리하기 때문에 인증 과정에서 예외가 발생하였음에도 모든 필터를 수행하기 때문에 비효율적이다.
	1. [이렇게 구현함으로써 해결할 수 있을 것 같다.](https://velog.io/@chullll/Spring-Security-JWT-%ED%95%84%ED%84%B0-%EC%A0%81%EC%9A%A9-2)



```
************************************************************

Request received for POST '/api/v1/members/oauth/kakao/login':

org.apache.catalina.connector.RequestFacade@2b0886a3

servletPath:/api/v1/members/oauth/kakao/login
pathInfo:null
headers: 
content-type: application/json
user-agent: PostmanRuntime/7.36.0
accept: */*
postman-token: 3fabb451-6ab9-4615-a81a-3f1ad8ded57a
host: localhost:8080
accept-encoding: gzip, deflate, br
connection: keep-alive
content-length: 89





************************************************************
```

```
************************************************************

Request received for GET '/api/v1/sse/subscribe':

SecurityContextHolderAwareRequestWrapper[ org.springframework.security.web.header.HeaderWriterFilter$HeaderWriterRequest@21f7221f]

servletPath:/api/v1/sse/subscribe
pathInfo:null
headers: 
host: localhost:8080
connection: keep-alive
sec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
accept: text/event-stream
sec-purpose: prefetch;prerender
sec-ch-ua-mobile: ?0
authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzb3VuOTk3QGdtYWlsLmNvbSIsImV4cCI6MTcwNDQ3MzA1MiwiaWF0IjoxNzAzODY4MjUyfQ.uLqG59Q3yhgqUv2C_lckr2GJzgfcepkG39xruNuzWN4
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
sec-ch-ua-platform: "Windows"
origin: http://localhost:3000
purpose: prefetch
sec-fetch-site: same-site
sec-fetch-mode: cors
sec-fetch-dest: empty
referer: http://localhost:3000/
accept-encoding: gzip, deflate, br
accept-language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,id;q=0.6,zh-CN;q=0.5,zh;q=0.4


Security filter chain: [
  DisableEncodeUrlFilter
  WebAsyncManagerIntegrationFilter
  SecurityContextHolderFilter
  HeaderWriterFilter
  CorsFilter
  LogoutFilter
  JwtAuthenticationEntryPoint
  JwtAuthenticationFilter
  RequestCacheAwareFilter
  SecurityContextHolderAwareRequestFilter
  AnonymousAuthenticationFilter
  SessionManagementFilter
  ExceptionTranslationFilter
  AuthorizationFilter
]


************************************************************
```

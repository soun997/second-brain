## Error Handling

### 인증 실패 핸들링

인증이 필요한 URI임에도 인증에 실패했다면(`Authentication`을 찾지 못했다면) `UNAUTHORIZED` 응답을 반환한다.
- 인증 헤더가 존재하지 않는 경우
- 인증 헤더에 Bearer Token이 존재하지 않는 경우

**Log**
```
[Time: 2024-01-04 07:53:16 | Class: StackTraceElement | Method: handleAccessDeniedException | LineNumber: 199]
[Exception: org.springframework.security.authentication.InsufficientAuthenticationException | Status: 401 UNAUTHORIZED | Message: Full authentication is required to access this resource]
```

**Response**
```json
{
    "errorCode": "AUTH_009",
    "description": "인증이 필요한 URI입니다."
}
```

### 인가 실패 핸들링

**Log**
```
[Time: 2024-01-04 09:19:23 | Class: StackTraceElement | Method: doFilter | LineNumber: 98]
[Exception: org.springframework.security.access.AccessDeniedException | Status: 403 FORBIDDEN | Message: Access Denied]
```

**Response**
```json
{
    "errorCode": "AUTH_010",
    "description": "Resource에 접근할 수 있는 권한이 없습니다."
}
```

### JWT Exception 핸들링

발생한 JWT Exception에 따라 다른 `ErrorCode`를 가진다.
- 인증 헤더와 Bearer Token은 존재하지만 유효하지 않은 경우
- JWT의 사용자 정보로 DB에 조회했을 때 사용자가 존재하지 않는 경

**Log**
```
[Time: 2024-01-04 07:59:16 | Class: StackTraceElement | Method: validateToken | LineNumber: 71]
[Exception: AUTH_001 | Status: 401 UNAUTHORIZED | Message: JWT 토큰이 만료되었습니다.]
```

```
[Time: 2024-01-04 07:59:07 | Class: StackTraceElement | Method: validateToken | LineNumber: 69]
[Exception: AUTH_003 | Status: 401 UNAUTHORIZED | Message: JWT 서명이 유효하지 않습니다.]
```

```
[Time: 2024-01-04 09:34:42 | Class: StackTraceElement | Method: lambda$loadUserByUsername$0 | LineNumber: 22]
[Exception: org.springframework.security.core.userdetails.UsernameNotFoundException | Status: 401 UNAUTHORIZED | Message: 사용자를 찾을 수 없습니다.]
```

**Response**
```json
{
    "errorCode": "AUTH_001",
    "description": "JWT 토큰이 만료되었습니다."
}
```
```json
{
    "errorCode": "AUTH_006",
    "description": "JWT 사용자 정보를 가져올 수 없습니다."
}
```
## Ref

https://colabear754.tistory.com/182
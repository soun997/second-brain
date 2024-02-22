```
Unable to load class named [io.jsonwebtoken.impl.DefaultJwtBuilder] from the thread context, current, or system/application ClassLoaders.  All heuristics have been exhausted.  Class could not be found.  Have you remembered to include the jjwt-impl.jar in your runtime classpath?
```

## 발생 이유

```groovy
// jwt  
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'  
compileOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'  
compileOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```

`io.jsonwebtoken:jjwt-impl`의 dependency 설정을 compileOnly로 설정했기 때문이다.
- 빌드 파일에는 해당 라이브러리가 포함되지 않는다. (참고 - [[Gradle Dependency Options]])

## 해결 방법

에러 메세지에서 볼 수 있듯이, `compileOnly`를 `runtimeOnly`로 변경해주면 된다.

```groovy
// jwt  
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'  
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'  
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```


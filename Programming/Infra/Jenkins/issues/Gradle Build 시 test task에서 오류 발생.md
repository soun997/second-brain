
## Gradle Build 시 :test에서 오류 발생

```
#14 78.01 VeganlifeApplicationTests > contextLoads() FAILED 
#14 78.01 java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:143 
#14 78.01 Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:801 
#14 78.01 Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:801 
#14 78.01 Caused by: org.springframework.beans.factory.BeanCreationException at AutowiredAnnotationBeanPostProcessor.java:498 
#14 78.01 Caused by: java.lang.IllegalArgumentException at PropertyPlaceholderHelper.java:180
```

몇 시간 째  위와 같은 오류와 싸우고 있는 중이다...

### 예상

`contextLoads()` 자체가 안되고 있고, 오류 트레이스를 보니 환경변수가 제대로 들어가지 않은 듯 하다.
현재 예상하기로는, test 환경에서 사용하는 yml이 제대로 정의되지 않았기 때문인 것 같다.

test 용 yml 이 없어서 `main/resource`에 있는 `application.yml`을 찾아갔지만, default로는 local로 설정되어 있었고 local에서는 h2 데이터베이스를 사용하는데, 이게 배포 환경에서는 제대로 작동이 안되는 것 같다.

### 발생 이유?

Spring의 profile을 dev로 설정해두었는데, jpa의 ddl-auto 정책을 `validate`로 해놨다.

그러나 현재 dev 환경의 DB에는 아무런 테이블 정보가 없기 때문에
**엔티티 클래스와 테이블이 정상적으로 매핑되지 않아, validation에 실패한 것이다.**

로컬 환경에서는 `application-local.yml`의 ddl-auto 정책을 `validate`로 바꿈으로써 해당 오류를 재현할 수 있다.

#### 그런데 말입니다...

![[Pasted image 20231209183610.png]]

여전히 오류가 난다.
- application-test.yml은 잘 읽어온다.
- docker compose에 환경변수를 하드코딩 해도 안된다.
	- 실질적인 빌드를 수행하는 Dockerfile에 환경변수가 제대로 전달이 안되는 것 같다.

### 해결

[[Dockerfile]]에 대한 기초 지식이 부족해서 벌어진 삽질기...

`ENV` 변수가 동적바인딩 되는 시점은 `docker run`, 실행 시점이고
`ARG` 변수가 동적바인딩 되는 시점은 `docker build`, 빌드 시점이다.

그렇기 때문에 빌드 시점에서 실행되는 테스트는 `ENV` 변수가 동적바인딩 되지 않았기 때문에 제대로 환경변수가 할당이 되지 않은 것이다.

**Build 시점에서는 `environment`를 이용해서 환경변수를 동적으로 설정해줄 수 없다.**
- Dockerfile에 `ARG`변수를 선언해주면  환경변수를 동적으로 받아와 Build 시점에 사용할 수 있다.

```Dockerfile
ARG MARIA_URL
ARG MARIA_USER
ARG MARIA_PASSWORD
```

### Ref

[배포 과정으로 인한 테스트 코드 작성의 불편사항 개선기](https://velog.io/@doli0913/%EB%B0%B0%ED%8F%AC-%EA%B3%BC%EC%A0%95%EC%97%90%EC%84%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C%EC%9D%98-%EB%B6%88%ED%8E%B8%EC%82%AC%ED%95%AD-%EA%B0%9C%EC%84%A0)
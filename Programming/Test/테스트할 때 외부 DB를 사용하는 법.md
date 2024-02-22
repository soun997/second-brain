## 서론

repository 레이어를 단위테스트할 때 MySQL DB를 사용하고 싶었다.
`application.yml` 파일에서 datasource를 MySQL 서버로 설정해주었다.
그러나 실행했을 때는 dataSource를 embedded DB로 replace했다는 로그만 찍혔다.
어떻게 하면 자동으로 Replace 하는 것을 멈출 수 있을까?

## 뜯어보기

먼저 단위테스트를 위해 사용한 `@DataJpaTest` 어노테이션을 확인했다.

``` java
// DataJpaTest.class
@TypeExcludeFilters({DataJpaTypeExcludeFilter.class})  
@Transactional  
@AutoConfigureCache  
@AutoConfigureDataJpa  
@AutoConfigureTestDatabase  
@AutoConfigureTestEntityManager  
@ImportAutoConfiguration  
public @interface DataJpaTest {
```

`@AutoConfigureTestDatabase` 어노테이션이 뭔가 수상하다.
테스트 DB를 자동으로 설정한다고?

``` java
// AutoConfigureTestDatabase.class
public @interface AutoConfigureTestDatabase {  
    @PropertyMapping(  
        skip = SkipPropertyMapping.ON_DEFAULT_VALUE  
    )  
    Replace replace() default AutoConfigureTestDatabase.Replace.ANY;  // 여기!
```

역시나 Replace와 관련된 프로퍼티가 존재했다.

> By default, ...They also use an embedded in-memory database (replacing any explicit or usually auto-configured DataSource). The [`@AutoConfigureTestDatabase`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html "annotation interface in org.springframework.boot.test.autoconfigure.jdbc") annotation can be used to override these settings.

[Spring Docs](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html)에서 관련된 내용을 확인할 수 있었다.
기본적으로는 in-memory DB를 사용하게 되어있어, datasource에 외부 DB를 설정해두어도 교체되는 것이었다.
그리고 아래에는 `@AutoConfigureTestDatabase`의 설정을 변경하여 이 설정을 override 가능하다고 나와있다.

`replace`의 값을 오버라이딩 해주면 되겠군!

## `AutoConfigureTestDatabase.Replace`

설정할 수 있는 값은 총 3가지였다.

`ANY`: 자동/수동 구성에 상관없이 replace
`AUTO_CONFIGURED`: 자동 구성일 경우에만 replace
`NONE`: replace 하지 않음

**Auto Configuration이 뭐지..?**
[스프링 부트의 Autoconfiguration 원리 및 만들어 보기](https://donghyeon.dev/spring/2020/08/01/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%EC%9D%98-AutoConfiguration%EC%9D%98-%EC%9B%90%EB%A6%AC-%EB%B0%8F-%EB%A7%8C%EB%93%A4%EC%96%B4-%EB%B3%B4%EA%B8%B0/)

우리는 datasource 자동 구성을 사용할 것이기 때문에 `NONE`으로 설정해주면 된다.
``` java
@AutoConfigureTestDatabase(replace = Replace.NONE)
```


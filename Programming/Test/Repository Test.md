
Repository Layer를 단위 테스트하는 방법을 알아보자.

## 무엇을 테스트해야 하나요?

1. DataSource의 설정이 정상적인지
2. JPA를 사용하여 데이터를 제대로 생성, 수정, 삭제하는지

## `@DataJpaTest`

테스트 시, JPA 관련 설정만 로드한다.
그렇기 때문에 `@SpringBootTest`보다 가볍다는 장점이 있다.

기본적으로 `@Entity` 어노테이션이 적용된 클래스들을 스캔하여 Spring Data JPA 저장소를 구성한다.

### 어노테이션 뜯어보기

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@BootstrapWith(DataJpaTestContextBootstrapper.class)  
@ExtendWith(SpringExtension.class)  
@OverrideAutoConfiguration(enabled = false)  
@TypeExcludeFilters(DataJpaTypeExcludeFilter.class)  
@Transactional  
@AutoConfigureCache  
@AutoConfigureDataJpa  
@AutoConfigureTestDatabase
@AutoConfigureTestEntityManager  
@ImportAutoConfiguration  
public @interface DataJpaTest {
```

#### `@Transactional`

테스트 메서드 하나하나가 `@Transactional`을 붙인 것처럼 동작한다는 것이다.
- 테스트가 완료되면 하나하나 롤백해줄 필요가 없다!
- ==그러나 같은 테스트 클래스 내에서의 Auto Increment 값은 초기화가 되지 않으니 주의하자!==
	- 테스트 메서드마다 DB를 새로 띄우지는 않는듯 하다.

#### `@AutoConfigureDataJpa`

`JpaRepository`에 대한 Auto Configuration을 사용한다.
- `JpaRepository`를 상속하여 구현한 인터페이스들을 `@Autowired`로 주입받을 수 있다.

#### `@AutoConfigureTestEntityManager`

`TestEntityManager`에 대한 Auto Configuration 을 사용한다.
- `TestEntityManager`를 Bean으로 등록하기 때문에 `@Autowired`로 주입받을 수 있다.

#### `@AutoConfigureTestDatabase`



## Ref

[@DataJpaTest](https://webcoding-start.tistory.com/20)

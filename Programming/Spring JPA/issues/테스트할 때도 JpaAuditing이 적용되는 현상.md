---
sticker: emoji//2705
---
## 문제 상황

**단위테스트 만을 수행했을 때는 문제없이 잘 동작한다.**

그러나 전체 테스트 케이스를 모두 돌렸을 때
Reflection을 이용해 createdAt을 잘 세팅해주었음에도 불구하고 `save` 메서드를 거치면 JpaAuditing이 적용되어 현재 DateTime을 기준으로 값이 들어가버린다.

## 누구 잘못이야?

단위 테스트만을 돌릴 때는 문제가 없었기 때문에
통합 테스트 어딘가에서 JpaAuditing을 실행하는 것 같다는 생각이 들었다.

**통합 테스트 클래스에 붙어있는 `@SpringBootTest`를 제거하면 잘 작동한다!**
이놈이 Configuration들을 가져와서 그런 것 같다.

## 왜?

### 확인 필요

`@SpringBootTest`는 기본적으로 모든 Configuration들을 가져온다.
그 중 `JpaAuditingConfig`를 로드하고 이를 캐싱하기 때문에 JpaAuditing이 적용되는 것 같다.

### 통합 테스트 실행 시

![[Pasted image 20231223064607.png]]

`jpaAuditingHandler`가 빈으로 등록된 것을 확인할 수 있다.

### 단위 테스트 실행 시

![[Pasted image 20231223064942.png]]

**`jpaAuditingHandler`가 존재하지 않는다!**

## 해결 방법

```java
@DataJpaTest  
@EnableJpaAuditing(setDates = false)  
public class MealLogRepositoryTest {
```

Repository 테스트 중 날짜를 설정해야 한다면 아예 `@EnableJpaAuditing`의 `setDates` 옵션을 꺼주면 된다.
이 옵션을 적용하면 자동으로 date를 설정하지 않는다.

## Ref

[스프링 테스트 - 테스트 컨텍스트 캐싱, @SpringBootTest, @WebMvcTest](https://cl8d.tistory.com/82)
[@DataJpaTest에서 자동으로 가져오는 configuration](https://docs.spring.io/spring-boot/docs/1.4.2.RELEASE/reference/html/test-auto-configuration.html)
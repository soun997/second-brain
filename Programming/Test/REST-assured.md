## REST-assured란?

우리는 스프링이 지원하는 테스트 어노테이션, JUnit5, Mockito 등을 사용하여 Layer 별로 [[단위 테스트]]를 진행했다.
그러나 단위테스트가 성공했다고 해서 외부 클라이언트와 연동했을 때 제대로 동작한다는 것을 보장할 수 없다.
단위 테스트로는 Layer들의 유기적인 결합, 일련의 과정까지 검증할 수 없다.
그렇기 때문에 ==요청부터 응답까지의 과정을 테스트할 수 있는 End to End 테스트의 도입==이 필요하다.

이럴 때 사용할 수 있는 것이 REST-assured 라이브러리이다.

## 그냥 Postman 쓰면 안돼?

물론 Postman을 사용해서 End to End 테스트를 진행할 수도 있다.

그러나 Postman은 일단 GUI 툴이기 때문에 Java 언어로 작성하는 테스트에 비해 유연성이 떨어진다.
특히 테스트 사전, 사후 동작을 지정하는 것도 REST-assured를 이용하는 것이 더 쉽다. (JUnit의 테스트 개념을 활용하기 때문이다)
- 각각의 테스트는 독립적으로 실행되어야 하므로 Database를 초기화해주어야 한다.
- Spring Security Filter를 통과하기 위해서는 access token을 발급해주어야 한다.

이곳에서 둘의 차이를 비교한 표를 확인할 수 있다.
[Rest-assured를 이용한 서비스 헬스체크 테스트 수행](https://blog.naver.com/genycho/221407018235)


## 테스트 격리시키기

각각의 테스트는 격리된 환경에서 실행되어야 한다.
그러나 E2E 테스트는 `@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)` 어노테이션을 설정해주기 때문에 `@Transactional`이 적용되지 않는다.
즉, 데이터베이스가 초기화되지 않는다는 뜻이다.

그러므로 직접 테스트 전에 데이터베이스 초기화를 실행해주도록 해야한다.

#### 방법
1. `@DirtiesContext`
	1. 컨텍스트 캐싱을 막아 항상 새로운 컨텍스트를 로드하도록 한다.
	2. 매번 컨텍스트를 로드하므로 시간이 많이 걸린다.
2. Repository 초기화
	1. E2E 테스트에 사용하는 repository를 알게 된다. (강하게 결합된다)
3. 쿼리를 이용한 초기화
	1. 최대한 블랙박스에 가까운 설정이다.

그러므로 사전에 native query를 날려 데이터베이스를 초기화 하는 방법을 선택하기로 했다.

참고한 글은 다음과 같다.
[테스트별로 DB 초기화하기](https://velog.io/@junho5336/%ED%85%8C%EC%8A%A4%ED%8A%B8%EB%B3%84%EB%A1%9C-DB-%EC%B4%88%EA%B8%B0%ED%99%94%ED%95%98%EA%B8%B0)
[@SpringBootTest의 테스트 격리시키기(TestExecutionListener), @Transactional로 롤백되지 않는 이유](https://mangkyu.tistory.com/264)

추가적으로, 테스트 시 사용하는 데이터베이스 설정에 맞게 native query를 작성해야 한다.

## Ref
[REST-Assured 알아보기 (테스트를 위한 클라이언트 객체)](https://loopstudy.tistory.com/427)
[RestAssured vs Mockmvc for unit and integration testing](https://stackoverflow.com/questions/49255965/restassured-vs-mockmvc-for-unit-and-integration-testing)
[ATDD와 함께 클린 API로 가는 길 3기 - 오리엔테이션](https://bgpark.tistory.com/149)
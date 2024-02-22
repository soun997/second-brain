---
sticker: emoji//2705
---
Repository Test를 하면서 `EntityManager`를 이용해 트랜잭션 관리를 해주려고 했다.
그런데 `fetch=FetchType.LAZY`로 설정해준 컬럼에 접근하려고 할 때 `LazyInitializationException`이 발생했다.
접근하려고 할 때 select 쿼리를 날려서 가져와주는 거 아니였나...?

괘씸해서 조금 더 파헤쳐보고자 한다.

## 환경

- 테스트 클래스에 `@SpringBootTest` 어노테이션이 붙어있다.
- `EntityManagerFactory`를 주입받아서 메서드마다 `EntityManager`를 생성하여 사용하였다.
	- [[Not allowed to create transaction on shared EntityManager]] 때문에 저러한 방식을 사용하였다.
- `getTransaction()`메서드를 사용해 `EntityTransaction`을 가져왔고 `begin()`, `commit()` 메서드를 통해 Transaction 처리해주었다.
- 저장, 조회 등의 연산은 해당 도메인의 `JpaRepository`의 연산을 사용했다.

## EntityManager에 대한 잘못된 이해

`EntityManagerFactory`를 사용해 직접 생성한 `EntityManager`는 공유되지 않는다.
그러나 `JpaRepository`는 공유된 `EntityManager`를 사용한다.
아예 다른 `EntityManager`을 사용했으니 Transaction 처리가 당연히 적용되지 않았으며, 이로 인해 `LazyInitializationException`이 발생했다.

## 해결 방법

`JpaRepository`를 사용하지 않고 직접 생성한 `EntityManager` 만으로 저장, 조회 로직을 수행하면, Transaciton이 적용되어 예외가 발생하지 않는다.

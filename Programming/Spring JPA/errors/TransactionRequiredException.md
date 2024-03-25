## 발생 이유

**트랜잭션이 아닌 곳**에서 **활성화된 트랜잭션을 필요로 하는 연산**을 호출하는 경우 발생한다.

## 비관적 락 적용 중 발생

비관적 락을 사용하면, `select for update` 연산을 실행하게 된다.
이는 `update` 연산을 위한 `select` 연산으로 보기 때문에 트랜잭션을 필요로 한다.

테스트 코드를 작성할 때 이 점을 유의하지 않아 오류가 발생한 적이 있다.

```java
@Test
void test() {

	// test codes...
	
	pet = petRepository.findByMemberId(member.getId()).get();
}
```

위의 `findByMemberId()` 메서드는 비관적 락을 적용한 상태이다.
검증을 위해 해당 메서드를 사용하여 엔티티를 조회하려고 했지만 **트랜잭션 밖에서 호출**했기 때문에 `TransactionRequiredException`이 발생하였다.


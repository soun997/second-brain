## `ReetrantLock`

재진입이 가능한 가장 일반적인 형태의 **배타락**이다.
- **재진입**이 가능하다는 말은, **이미 락을 획득한 쓰레드가 자신이 보유한 락과 동일한 락을 다시 요청할 수 있다는 것을 의미**한다. ➡️ 락의 획득이 호출 단위가 아닌 스레드 단위로 일어난다.
- 만약 메서드의 호출 흐름이 A ➡️ B와 같고, A, B 둘 다 `synchornized` 메서드 블록이라면
	- **A에서 lock을 획득했다면, B 진입 시 락을 얻기 위해 대기할 필요가 없다!**

임계 영역의 공유자원을 읽거나 쓰기 위해서는 lock을 반드시 보유하고 있어야 한다.

Ref: [<CS 지식> Java 고유락 (Intrinsic Lock)](https://velog.io/@kimmy/CS-%EC%A7%80%EC%8B%9D-Java-%EA%B3%A0%EC%9C%A0%EB%9D%BD-Intrinsic-Lock)


## `ReetrantReadWriteLock`

읽기에는 공유적이고, 쓰기에는 배타적인 락이다.
- 즉, 읽기를 위한 lock과 쓰기를 위한 lock을 따로 제공한다.
- 현재 쓰레드 읽기 lock + 다른 쓰레드 읽기 lock ➡️ ✅
- 현재 쓰레드 읽기 lock + 다른 쓰레드 쓰기 lock ➡️ ❌


## `StampedLock`

`ReentrantReadWriteLock`에 **낙관적인 lock의 기능들을 추가**한 락이다.
- lock을 걸거나 해지할 때, stamp(`long` 타입의 정수값)를 사용한다.

**일반적인 읽기 lock**이 걸려있다면, 쓰기 lock을 얻기 위해서는 읽기 lock이 풀릴 때가지 대기해야 한다.
**낙관적인 읽기 lock**이 걸려있다면, 쓰기 lock에 의해 바로 풀리게 된다.
- 낙관적 읽기에 실패했다면, 읽기 lock을 얻어서 다시 읽어와야 한다.

`StampedLock` 클래스는 쓰기와 읽기가 충돌할 때만 쓰기가 끝난 후에 읽기 lock을 건다.

위의 lock들과는 다르게 재진입이 불가하다는 특성이 있다.

참고: [[낙관적 락]], [[비관적 락]]

### stamp

JPA에서의 낙관적 락을 위한 `@Version` 필드를 생각하면 된다.
lock을 획득할 당시의 stamp와 현재의 stamp가 다름에도 unlock을 시도하면 예외가 발생한다.


## Ref

[쓰레드의 동기화 #2 Lock과 Condition을 이용한 동기화](https://yonghwankim-dev.tistory.com/472)
[concurrent programming - StampedLock](https://rightnowdo.tistory.com/entry/JAVA-concurrent-programming-StampedLock)
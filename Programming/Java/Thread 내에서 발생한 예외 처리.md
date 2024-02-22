## 실패한 예외 처리

Thread에서 발생한 예외를 처리하기 위해 `assertThatThrownBy()`로 해당 스레드를 실행하는 메서드를 감쌌다.

```java
assertThatThrownBy(() -> {  
    threadA.start();
    threadB.start();
    sleep(3000);  
})
.as("두 개의 스레드가 동시에 좋아요 add 메서드를 호출했을 경우 동시성 이슈가 발생한다.") 
.isInstanceOf(SqlException.class);
```

그러나 예외가 발생하지 않았다는 `AssertionError`가 발생했다.

### 왜?

Java에서 각 스레드는 독립적으로 실행되기 때문에, 하나의 스레드에서 발생한 예외는 다른 스레드에 직접적으로 전파되지 않는다.

그렇기 때문에 메인 스레드에서 파생된 다른 스레드들에서 예외가 발생하더라도 메인 스레드와는 독립적으로 발생한 것으로 간주하기 때문에 따로 처리하지 않는다.

## 두번째 시도


## Ref

[스레드(Thread) 이해하기 -1 : 구조, 상태, 예시](https://adjh54.tistory.com/167) - 메인 스레드가 가장 먼저 종료된다.
[Java Thread 내에서 발생한 Exception 처리](https://github.com/HomoEfficio/dev-tips/blob/master/Java-Thread%EB%82%B4%EC%97%90%EC%84%9C-%EB%B0%9C%EC%83%9D%ED%95%9C-Exception-%EC%B2%98%EB%A6%AC.md)
[Thread 안에서 발생하는 예외는 어떻게 처리되나](https://studyandwrite.tistory.com/536)
[재고시스템으로 알아보는 동시성이슈 해결방법](https://thalals.tistory.com/370) - `CountDownLatch` 사용 방법
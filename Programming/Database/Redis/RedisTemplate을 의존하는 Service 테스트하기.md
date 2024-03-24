## 서론

RedisTemplate을 사용하는 Service Layer를 단위 테스트하려고 했지만, 이를 mocking하는 과정에서 어려움을 겪었다.

- `RedisTemplate`의 `opsForValue()`가 `null`이라는 오류 (`NullPointerException`)
- `opsForValue()`까지 mocking하기 위해선 어떻게 해야하지?

## 해결

### RedisTemplate, ValueOperation Mocking

```java
@Mock RedisTemplate<String, Long> ploggingRedisTemplate;
@Mock ValueOperations<String, Long> valueOperations;
```

### mocking 객체 반환값 넣어주기

```java
given(ploggingRedisTemplate.opsForValue()).willReturn(valueOperations);
given(ploggingRedisTemplate.opsForValue().decrement(TotalPlogger.KEY)).willReturn(total - 1L);
```

### 설명

RedisTemplate 뿐만 아니라

RedisTemplate의 메서드인 `opsForValue()`의 반환값인 ValueOperations까지 mocking해준다.

그래야만 `opsForValue()`의 `decrement()` 메서드를 사용할 수 있다.

## Ref

[https://stackoverflow.com/questions/43303212/mock-redis-template](https://stackoverflow.com/questions/43303212/mock-redis-template)
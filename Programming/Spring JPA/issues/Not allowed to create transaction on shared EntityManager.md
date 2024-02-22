---
sticker: emoji//2705
---
## 발생 이유

`EntityManager`를 클래스 내에서 `@AutoWired`로 주입받아 사용하려고 했다.
테스트하면서 트랜잭션 처리를 해줄 부분이 있어 `getTransaction()` 메서드를 사용해 `EntityTransaction` 객체를 가져오고자 했다.

그러나 Spring Data JPA는 공유 `EntityManager` 인스턴스로 직접 트랜잭션을 관리하는 것을 허용하지 않는다.

`Autowired`로 주입한다는 것은 Bean으로 등록된 `EntityManager`를 사용한다는 것일 텐데...
그럴거면 왜 Bean으로 등록하는 거지?


## 디버깅으로 파헤치기

### `SimpleJpaRepository`

디버깅을 통해 EntityManager에 어떤 값이 들어가는지 알아보자.

#### 주입받은 `EntityManager`

![[Pasted image 20231219062354.png|1000]]
![[Pasted image 20231219062924.png]]
#### `Repository1`

![[Pasted image 20231219062442.png|1000]]
![[Pasted image 20231219062926.png]]
#### `Repository2`

![[Pasted image 20231219062505.png|1000]]

![[Pasted image 20231219062935.png]]

모두 같은 `EntityManagerFactory`에서 나왔다는 것을 알 수 있다.
또한, `EntityManager` 인터페이스를 사용하고 있다.

그러나 주입받은 `EntityManager`와 아래의 repository들은 다른 `EntityManager` 프록시를 공유하고 있다.



### 에러 메세지 따라가기

#### `SharedEntityManagerCreator.java`

```java
case "getTransaction":  
    throw new IllegalStateException(  
          "Not allowed to create transaction on shared EntityManager - " +  
          "use Spring transactions or EJB CMT instead");
```

[[Dynamic Proxy]] 방식을 이용한다.

`EntityManager` 인터페이스의 메서드를 실행하면, 실행한 메서드의 이름에 따라 특정 동작을 하도록 구현되어 있다.

에러 메세지를 내뱉는 곳을 따라가면 위의 코드와 같다.
`getTransaction()` 메서드는 Shared `EntityManager`에서 허용되지 않는 연산이다.

프록시 패턴을 사용하여 접근 제한을 구현한 예라고 생각하면 될 것 같다.


### 그럼 Transaction은 어떻게 처리하는데?

#### `JpaTransactionManager.java`

```java
@Override  
protected void doBegin(Object transaction, TransactionDefinition definition) {  
    JpaTransactionObject txObject = (JpaTransactionObject) transaction;  

	...
	
    try {  
       if (!txObject.hasEntityManagerHolder() ||  
             txObject.getEntityManagerHolder().isSynchronizedWithTransaction()) {  
          EntityManager newEm = createEntityManagerForTransaction();  
          if (logger.isDebugEnabled()) {  
             logger.debug("Opened new EntityManager [" + newEm + "] for JPA transaction");  
          }  
          txObject.setEntityManagerHolder(new EntityManagerHolder(newEm), true);  
       }
    ...
```

Transaction을 위한 새로운 EntityManager를 생성한다.

```java
* Create a JPA EntityManager to be used for a transaction.  
* <p>The default implementation checks whether the EntityManagerFactory  
* is a Spring proxy and delegates to  
* {@link EntityManagerFactoryInfo#createNativeEntityManager}  
* if possible which in turns applies  
* {@link JpaVendorAdapter#postProcessEntityManager(EntityManager)}.  
* @see jakarta.persistence.EntityManagerFactory#createEntityManager()  
*/
protected EntityManager createEntityManagerForTransaction() {  
    EntityManagerFactory emf = obtainEntityManagerFactory();  
    Map<String, Object> properties = getJpaPropertyMap();  
    EntityManager em;  
    if (emf instanceof EntityManagerFactoryInfo emfInfo) {  
       em = emfInfo.createNativeEntityManager(properties);  
    }
    ...
```


![[Pasted image 20231219084019.png|1000]]

`Autowired`를 통해 주입받은 `EntityManager`와 `EntityManagerFactory`를 통해 생성한 `EntityManager`의 차이가 보이는가?

직접 생성해준 `EntityManager`는 `SessionImpl`의 프록시 객체이다.
그렇기 때문에 위와 같은 접근 제한이 없기에 `getTransaction()`메서드를 사용할 수 있었다.

## 결론

`@Autowired`를 통해 주입받은 `EntityManager`는 `SharedEnttiyManagerCreator`를 통해 생성된다.
- 프록시 객체의 `getTransaction()`에 대한 접근 제한이 걸려있기 때문에 해당 연산을 사용하면 에러가 발생했다.

이에 비해 `EntityFactoryManager`를 통해 생성한 `EntityManager`는 이러한 접근제한이 없다.
그렇기 때문에 `EntityManager`의 메서드들을 자유롭게 사용할 수 있다.


## Ref

[대체 JPA에서 Proxy로 초기화된다는 말이 뭔데?](https://happy-coding-day.tistory.com/entry/%EB%8C%80%EC%B2%B4-JPA%EC%97%90%EC%84%9C-Proxy-%EB%A1%9C-%EC%B4%88%EA%B8%B0%ED%99%94%EB%90%9C%EB%8B%A4%EB%8A%94-%EB%A7%90%EC%9D%B4-%EB%AD%94%EB%8D%B0)

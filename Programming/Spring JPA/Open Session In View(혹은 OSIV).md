### Open Session In View 란?

> 관례상 OSIV라고 한다. 앞으로는 OSIV로 표현을 통일하도록 하겠다.


OSIV는 영속성 컨텍스트의 생존 범위를 설정할 수 있는 속성이다.

##### `true`일 경우 (Default)

![[Pasted image 20231123163239.png]]

영속성 컨텍스트가 트랜잭션 범위를 넘어서까지 살아있게 된다.
- API라면 클라이언트에게 응답될 때까지, View라면 View가 렌더링 될 때까지 살아있다.

##### `false`일 경우

![[Pasted image 20231123163248.png]]

영속성 컨텍스트의 수명은 트랜잭션 범위를 넘지 않는다.


### OSIV + [[GraphQL]]

GraphQL을 사용할 때, n+1 문제를 해결하기 위해 `DataLoader`를 사용하는 경우가 잦다.

``` java
query GetProductDetailPageData($id: ID!) {  
	product {  
		product(id: $id) {  
			//DataLoader를 사용하지 않는 부분.(A)  
			id  
			title  
			...  
			//DataLoader를 사용하는 부분.(B)  
			productInfo {  
			...  
			}  
		}  
	}  
}
```

(B)를 조회할 때, `DataLoader`는 새로운 thread를 만들고 새로운 connection을 사용해야 한다.

그러나 OSIV를 사용한다면, (A) 조회 후에도 connection이 반납되지 않기 때문에
하나의 query를 실행하는 데에 2개의 connection이 필요하게 된다.

> ✅ [이는 DB connection의 데드락을 발생시켜 서버를 죽일 수도 있다!](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)


### OSIV + [[Lazy Loading]]

OSIV를 사용하지 않을 경우, 트랜잭션이 commit되면 `EntityManager` 또한 함께 닫히게 된다.

만약 트랜잭션 외부에서 Lazy Loading을 사용하면 `no Session`이라는 예외가 발생하게 된다.

> 그러므로 OSIV를 사용함으로써 발생하는 문제들을 해결하기 위해서 
> 무작정 OSIV 속성을 `false`로 바꿔서는 안된다.


### 결론

#### OSIV를 true로 한다?

**장점**
- 영속성 컨텍스트를 유지하기 때문에 Lazy Loading을 사용할 수 있다.

**단점**
- 실시간 트래픽이 중요한 어플리케이션에서는 DB Connection이 모자를 수 있다.
- 하나의 API 호출 시에, 여러 개의 connection을 사용할 경우 DB Connection의 데드락이 발생할 수 있다.

#### OSIV를 false로 한다?

**장점**
- 트랜잭션이 commit 될 때, 영속성 컨텍스트가 소멸하므로 connection 최적화가 가능하다.

**단점**
- Lazy Loading을 사용할 경우 `no Session` 예외가 발생할 수 있다. (정확히 어떤 상황에서 발생?) ^1d97d2


## Ref
[open-session-in-view 를 알아보자](https://gracelove91.tistory.com/100)
[Spring Boot의 open-in-view, 그 위험성에 대하여.](https://medium.com/frientrip/spring-boot%EC%9D%98-open-in-view-%EA%B7%B8-%EC%9C%84%ED%97%98%EC%84%B1%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC-83483a03e5dc)

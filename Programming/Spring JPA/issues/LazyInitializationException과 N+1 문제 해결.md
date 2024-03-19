---
sticker: emoji//2705
---
## 문제 상황

Service 레이어에서 엔티티를 반환하고 Controller에서 이를 Response DTO로 매핑해주었다.
그러나 매핑하는 과정에서 자식 엔티티의 프록시 객체에 접근할 때 LazyInitializationException이 발생했다.

## 발생 이유

OSIV 설정을 false로 해두었기 때문에 영속성 컨텍스트의 생존 범위가 트랜잭션 범위로 한정되었다.
그러므로 모든 지연 로딩을 트랜잭션 안에서 처리해야 한다.

## 해결 방법

### 실패한 해결 방법

**에러가 발생하는 조회 메서드에 한정하여 fetch join을 이용해 지연로딩을 강제로 호출해준다.**

그러나 문제가 되는 엔티티는 `@OneToMany` 연관관계를 두 개 가지고 있어 fetch join을 두 번 사용할 수 없었다.
- [[MultipleBagFetchException]]이 발생한다.

**Transactional 안에서 자식 엔티티들을 영속화한다.**

제대로 원하는 대로 제대로 동작한다.
다만 자식 엔티티를 영속화하기 위해 조회하는 순간 N+1 문제가 발생한다.

N+1 문제를 어떻게 해결할 수 있을까?

### default_batch_fetch_size 를 이용한 해결

```yaml
spring:  
  jpa:  
    open-in-view: false  
    properties:  
      hibernate:  
        default_batch_fetch_size: 1000
```

다음과 같이 `application.yml` 파일에 설정해준다.

batch_fetch_size를 설정해두면 조회 결과가 N개일 때 각각 쿼리를 날리는 것이 아니라, **default_batch_fetch_size**만큼 한꺼번에 날리게 된다.
- N+1 문제를 해결할 수 있다.



[Vegan Life 프로젝트 내에서의 성능 개선](https://github.com/Vegan-Life/VeganLife-Backend/pull/197)

## Ref
[JPA - OSIV(Open Session In View) 정리](https://ykh6242.tistory.com/entry/JPA-OSIVOpen-Session-In-View%EC%99%80-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94)
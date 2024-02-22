---
sticker: emoji//1f914
---

## 고민

Service에서 다른 Repository를 의존하는 코드
Service에서 다른 Service를 의존하는 코드
Controller에서 다른 Service를 의존하는 코드

프로젝트를 진행하면서 이처럼 다른 레이어의 Service나 Repository가 호출해야 하는 경우를 발견할 수 있었다.

각각의 방식의 장단점을 알아보고 프로젝트에 맞게 적용해보자.

## Service에서 다른 Repository를 의존

하나의 Service가 여러 Repository를 의존한다.

단순한 기능(CRUD)을 필요로 한다면 크게 문제되지 않는다.

그러나 프로젝트가 복잡한 기능을 요구하는 경우 Service의 책임감이 무거워진다.
(아래는 실제 프로젝트에서 발췌한 코드이다)

![[Pasted image 20231213000451.png|600]]

또한, Service에 구현한 복잡한 기능을 여러 곳에서 사용하고 싶어도 재사용할 수 없기 때문에 중복되는 코드가 늘어난다.


## Service에서 다른 Service를 의존

다른 방법으로는 하나의 Service가 여러 Service를 의존하는 방법이 있다.

같은 Layer 간 참조 관계를 가지기 때문에 **순환 참조가 발생할 수 있다**
- 이는 곧 참조 방향이 한쪽으로 흐르고 있지 않음을 의미한다.
- +) 순환 참조가 발생하는 것 자체가 설계에 문제가 있다고 주장하는 사람들도 있다.

### 순환 참조를 방지하는 방법?

서비스들의 의존성 방향을 한쪽으로 고정할 순 없을까?

물론 할 수 있다.,
Service를 상위 계층(Component Service)과 하위 계층(Module Service)로 분리하는 것이다.


### 너무 많은 Service를 의존하게 된다.

지금까지 구현했던 Service는 Repository와 1 : 1 매핑이 되지 않았다.
다양한 도메인의 Service와 의존하고 있었던 것이다.

그렇기 때문에 메서드를 재사용하기 어려웠고 생산성 또한 떨어지는 것을 느꼈다.

[Service에서 다른 Service를 의존하게 된다면 ?🌟](https://taesan94.tistory.com/268)

이 게시글와 나의 생각이 어느정도 일치하는 것 같다. 너무 좋은 글!






- [ ] Transaction 처리에 문제가 없나? ([[Transaction 전파]])




## Ref

[BE | Service에서 Service를 의존할까 Repository(Dao)를 의존할까 | 2022.05.24](https://github.com/woowacourse/retrospective/discussions/15)
[Service에서 다른 Service 의존에 대하여](https://jangjjolkit.tistory.com/62)
[스프링부트 서비스 계층(Service Layer)을 분리해보자!](https://velog.io/@pgmjun/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B3%84%EC%B8%B5Service-Layer%EC%9D%84-%EB%B6%84%EB%A6%AC%ED%95%B4%EB%B3%B4%EC%9E%90)

---
sticker: emoji//2705
---
## 서론

정말 이 고민은 해도해도 끝이 없는 것 같다.
매번 이에 대한 생각이 들 때마다 사색에 잠기는 것 보다는
나만의 기준을 만들고 나중에도 비슷한 고민이 들었을 때 참고하고자 문서로 남긴다.

결론을 먼저 말하자면, 모든 설계는 트레이드 오프의 산물이라는 것이다.
(참고로 Layered Architecture를 사용한다는 가정 하에 글을 작성했다)

## DTO란?

이 주제에 대한 정답을 찾기 위해서는 [[DTO]]가 진정 무엇을 위한 것인지 생각해 볼 필요가 있었다.

여러 글들을 찾아보며 내가 DTO의 목적보다는 구조적인 통일성에 더 집착하고 있었음을 깨달았다.
- 항상 트레이드 오프가 필요하다는 것을 잊지 말자

DTO를 분리하는 목적은 단순하다.
> ‘수평 방향의 책임을 나눠 god class를 예방하자’

![[Pasted image 20240107054441.png]]

위 그림의 첫번째 사례와 같이 여러 Layer에 걸친 책임을 가지게 된다면 이는 곧 God Class가 됐음을 의미한다.
DTO는 이러한 책임을 나눠 가져가는 객체인 것이다.

## 어디서 어떻게 변환할 것인가?

다음과 같은 경우의 수가 존재했다.

1. Controller 레이어에서 DTO를 Entity로 변환, 이를 Service 레이어로 넘긴다.
2. Controller 레이어에서 DTO를 Service 레이어로 넘기고 그곳에서 Entity로 변환한다.
3. Service 레이어에서 Entity를 DTO로 변환, 이를 Controller 레이어로 넘긴다.
4. Service 레이어에서 Entity를 Controller 레이어로 넘기고 그곳에서 DTO로 변환한다.


하나하나 어떠한 장단점이 있는지 알아보자.
##### Controller 레이어에서 DTO -> Entity 혹은 Entity -> DTO
- Controller가 Domain에 의존한다.
- Service 레이어가 DTO에 의존하지 않는다.
	- 그렇기 때문에 Service 레이어의 메서드들을 재사용하기 좋다.

##### Service 레이어에서 DTO -> Entity 혹은 Entity -> DTO
- Service 레이어에 DTO가 의존한다.
- Controller가 Domain에 의존하지 않는다.
	- Domain의 비즈니스 로직이 혼재될 위험이 적다.


의존성에 대한 관점에서 위의 경우들을 다시 살펴보자.
##### Domain에 의존한다.
- 이는 곧 도메인의 비즈니스 로직에 접근할 수 있다는 것을 의미한다.
- **Domain에 의존할 필요가 없는 Controller Layer에서 Domain에 의존하게 된다면, 이는 곧 해당 Layer에 비즈니스 로직이 혼재될 위험성이 생긴다.** (DTO 변환 이외의 작업을 수행할 수 있다.)
	- 신입 개발자가 우리 프로젝트에 참여했을 때의 상황을 가정해보자. 그들은 어떻게 행동할까?
	- 그러나 이런 경우가 많이 발생할까? 라는 의문이 들긴 한다.

##### DTO에 의존한다.
- 이는 곧 Presentation의 변화에 의존하게 된다는 것을 의미한다.
- **Service Layer가 DTO에 의존하게 된다면 Presentation의 변화는 Service 레이어까지 전파될 것이다.**
	- 특히 Presentation Layer는 화면에 보여지는 데이터를 다루는 만큼, 자주 변경될 것이다.
	- 또한 DTO의 형식에 따라 Service Layer의 메더드들은 계속 늘어갈 것이다. (뚱뚱한 Service)


## 나의 선택은

>애초에, SearchCondition 계열이나, presentation DTO -> external DTO로 바로 매핑해야 하는 케이스는 presentation DTO가 service layer 파라미터로 전달 되는 것이 자연스럽다. 
>따라서 변환에 대한 책임을 강제하기 보다는, 상황에 맞게 유연하게 선택하는 것이 좋아보인다.

상황에 따라서 DTO가 전달되는 것이 자연스러울 때도 있고, Domain이 전달되는 것이 자연스러울 때도 있다.
나는 구조적인 통일성에만 집중하여 무조건 하나로 통일해야 한다고 생각했다. (물론 일관성을 챙김으로써 얻는 이점도 분명히 있을 것이다)

이제는 조금 생각이 바뀌었다.

최종적으로 정리를 하자면
1. Controller -> Service의 경우에는 DTO를 전달한다.
2. Service -> Controller의 경우에는 Entity를 전달한다.
3. 트레이드 오프를 고려했을 때 위의 규율을 따르지 않을 수도 있다. 이를 허용하나 반드시 주석으로 타당한 설명을 달아둔다.

그 이유는 다음과 같다.
1. Controller -> Service의 경우에는 DTO를 전달한다.
	1. Controller에서 Service로 넘기는 DTO는 등록, 삭제, 수정을 위한 내용이거나 Search Condition일 것이다.
	2. 등록, 삭제, 수정을 위한 DTO나 Search Condition은 조합이 한정적이고 잘 변화하지 않는다.
2. Service -> Controller의 경우에는 Domain을 전달한다.
	1. Presentation Layer는 곧 애플리케이션에서 보여주어야 할 데이터들을 응답하게 된다.
	2. 해당 데이터들의 조합은 무수히 많아질 수 있으며 조합마다 Service에 메서드를 구현하는 것은 비효율적이라고 생각하기 때문이다.
	3. 단, `LazyInitializationException` 등의 예외를 해결하기 위해 DTO를 반환해야 하는 경우도 생길 수 있다. 이런 경우에는 DTO를 반환할 수 있다.


**물론 현재의 결론은 언제든지 변화할 수 있으므로 유연하게 생각하는 자세를 가져야 한다.**


## Ref

[DTO의 객체변환은 어느 계층에서 하는것이 적절할까?](https://anaog.tistory.com/7)
[MVC Layered Architecture - DTO 전달/변환/파라미터 설계](https://umbum.dev/1208/)
[DTO는 어떤 Layer에 포함되어야 하는가.](https://jiwondev.tistory.com/251)
[Dto 사용시기에 대한 질문](https://www.inflearn.com/questions/139564/dto-%EC%82%AC%EC%9A%A9%EC%8B%9C%EA%B8%B0%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A7%88%EB%AC%B8)
[Controller에서 Service에 값을 어떻게 전달할까?](https://kafcamus.tistory.com/12?category=912020)
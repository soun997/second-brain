## 서론

현재 EC2 t3a.medium 인스턴스를 사용하고 있는데, 월간 약 50000원 이상의 요금이 청구되었다.
사이드 프로젝트이기도 했고, 수익을 전혀 내지 않고 있기 때문에 팀원들과 분담하더라도 큰 금액이었다.

이에 EC2 인스턴스 다운그레이드를 검토하던 중, AWS Lambda와 같은 서버리스 아키텍처 또한 고려할 만 하다는 판단에 조사를 하게 되었다.

> 현재 구축 중인 서비스 개요
> 언어: Java17
> 프레임워크: Spring Boot 3


## 선택

결론적으로 인스턴스 다운그레이드를 하기로 결정하였다.
그 이유는 아래와 같다.
### Java와 서버리스 아키텍처 조합은 좋지 않다?

#### 시작 시간 및 초기화 지연

[[Java는 왜 느릴까]]
Java 애플리케이션의 실행을 위해서는 JVM의 로드가 필수적이다.
또한, 초기화 단계에서 컴파일링 과정이 포함되기 때문에 많은 시간을 소요하게 된다. 

서버리스 아키텍처의 경우에는 요청이 발생했을 때 인스턴스를 활성화하고, 일정시간 사용하지 않으면 다시 비활성화 한다.
이러한 특성 때문에 서버리스 함수가 처음 호출될 때 초기화 지연이 발생하게 되는데, Java 애플리케이션의 이런 특성으로 인해 그 시간이 더욱 늘어날 수 있다. (**콜드 스타트 문제**)

콜드 스타트 문제는 사용자 경험을 저해시키고, 비용을 증가시킬 수 있다.
- 자주 사용되는 함수의 핫 스타트, Pre-warming된 인스턴스 사용 등의 해결 방법이 있다.

### 서버리스 아키텍처 도입의 오버헤드

아직 개발 중인 서비스이기 때문에, 최대한 빠르게 수정하고 다시 서버를 정상적으로 실행해야 했다.


## 다음 이야기

[[EC2 다운그레이드]]
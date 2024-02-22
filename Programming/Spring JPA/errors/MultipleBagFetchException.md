2개 이상의 `@OneToMany` 연관관계 테이블에 대하여 fetch join(혹은 Eager Loading)을 사용했을 때 발생하는 오류이다.

## 뭐가 문제인데?

### Cartesian Product 발생

위의 상황에서 2번 이상의 fetch join이 실행되면 Collection의 Cartesian Product가 발생할 위험이 있다.
- Cartesian Product는 모든 가능한 행들의 조합을 반환하는 현상이다.

보통의 DBMS들은 join을 [[NL 조인]]으로 처리한다.
- NL 조인은 중첩 for-loop와 비슷한 수행 구조를 가진다.
- NL 조인은 [[인덱스]]를 활용한 조인이다.

만약 인덱스 튜닝이 제대로 되지 않았다고 하자 (JPA는 Index Hint 지원이 되지 않는다, Native Query 사용 필요)
만약 등치(=) 조건이 아닌 조건으로 조인을 수행한다면?
충분히 예상치 못한 Cartesian Product가 발생할 수 있는 상황이다.

**그렇기 때문에 예상치 못한 Cartesian Product 발생 위험을 염두에 두고 해당 Exception을 구현해둔 것 같다.**

## 해결 방법

해당 예외를 해결하기 위해서 다음과 같은 방법을 시도할 수 있다.
- 자식 테이블 중 하나에만 fetch join을 건다.
- 모든 자식 테이블을 Lazy Loading하도록 한다.
	- 위의 방법들은 OSIV 설정에 따라 LazyInitializationException을 발생시킬 수도 있다.
- List가 아닌 Set으로 엔티티들을 받아온다.
	- 절대절대절대 좋은 방법이 아니다!, 그저 예외가 발생하지 않도록 눈속임일 뿐,,, 성능은 최악이다.
- Fetch Join을 나누어서 실행한 후에 조합한다.
- **BatchSize 옵션을 이용하여 컬렉션들을 지연로딩한다.**

나는 위의 해결 방법 중 김영한 강사님께서 언급하시기도 한 BatchSize 옵션을 이용하여 해결해보려고 한다.
[[또 LazyInitializationException...]]

## Ref

[Fetch Join 할 때 MultipleBagFetchException 해결하는 법](https://devlog-wjdrbs96.tistory.com/421)
[Fetch Join의 한계](https://velog.io/@antcode97/Fetch-Join%EC%9D%98-%ED%95%9C%EA%B3%84)
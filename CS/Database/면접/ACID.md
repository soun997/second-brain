>데이터의 유효성을 보장하기 위한, [[Transaction]]의 특징들을 가리키는 말이다.

## ACID

### Atomicity(원자성)

트랜잭션 내의 모든 작업은 반영되거나 롤백된다.

### Consistency(일관성)

데이터는 미리 정의된 규칙에서만 수정이 가능하다.
- 숫자형 컬럼에 문자열 값을 저장할 수 없다.

### Isolation(고립성)

A와 B 두 개의 트랜잭션이 실행되고 있을 때, A의 작업들이 B에게 보여지는 정로를 의미한다.

### Durability(영구성)

한 번 반영된 트랜잭션의 내용은 영원히 적용된다.


## Ref

[Transaction과 ACID란?](https://chrisjune-13837.medium.com/db-transaction-%EA%B3%BC-acid%EB%9E%80-45a785403f9e)
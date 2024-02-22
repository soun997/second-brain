## 서론

보통 조회를 할 때, DB 테이블의 모든 데이터를 조회하는 경우는 드물다.
이 때 우리는 페이징을 사용하여 원하는 개수만큼의 레코드만 조회할 수 있다.

Spring Data JPA에서는 `Pageable` 객체를 JPA Query의 매개변수로 넣어주기만 해도 자동으로 pagination query를 날려준다.

어떻게 이게 가능한 것일까??

## Spring Data JPA에서의 페이징

[Pageable in Spring JPA](https://medium.com/@avocadi/pageable-in-spring-jpa-84e75e0c69f1)

Spring Data JPA에서 pagination을 적용하기 위해
1. `Pageable` 객체를 repository method의 마지막 인수로 전달한다.
2. repository method의 return type을 지정한다.

### 1. `Pageable` 인스턴스를 repository method의 마지막 인수로 전달한다.

Spring Data JPA는 `Pageable`인스턴스를 파싱하여 pagination query를 자동 생성한다.
- 어떻게?????

### 2. repository method의 return type을 지정한다.

return type은 크게 3가지가 있다. ([[Page와 Slice]])
1. `Page`
2. `Slice`
3. `List`




## Ref

[Pageable in Spring JPA](https://medium.com/@avocadi/pageable-in-spring-jpa-84e75e0c69f1)
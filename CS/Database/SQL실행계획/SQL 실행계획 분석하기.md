## MySQL Explain 항목별 의미

**id**
- 실행순서, SQL 문이 실행되는 순서

**table**
- 어떤 테이블에 대한 접근을 표시하고 있는지 나타낸다.

**select_type**
- SELECT 문의 유형
- 대부분 SIMPLE - 내부 쿼리가 없을 경우
- PRIMARY - 서브쿼리를 감싸는 외부 쿼리 혹은 UNION이 포함된 SQL문의 첫번째 SELECT
- SUBQUERY - 스칼라 서브쿼리, WHERE 절의 중첩 서브쿼리
- DEPENDENT SUBQUERY - UNION, UNION ALL을 사용하는 서브쿼리가 메인 테이블에 영향을 받는 경우

**type**
- 접근 방식, 대상 테이블로의 접근이 효율적인지 판단 가능
- const
	- 고유 인덱스나 기본키를 사용하여 단 1건의 데이터만 접근, 성능 굿
- eq_ref
	- 조인 시 드라이빙 테이블이 드리븐 테이블에 접근하여 고유 인덱스나 기본키를 사용해 단 1건의 데이터를 조회 - 조인 키가 드리븐 테이블에 유일, 조인 시 성능 굿
- ref
	- 조인 시 드라이빙 테이블이 드리븐 테이블에 접근하는 데이터가 2개 이상인 경우 혹은 where 절의 비교 연산자
	- 드라이빙 - 드리븐 테이블은 일대다 관계
- ref_or_null
	- ref와 비슷하나 ISNULL 구문 수행 시 인덱스를 사용하는 경우
- range
	- 테이블 내 연속된 데이터를 조회하는 경우(비교 연산자, ISNULL, BETWEEN, IN 등의 범위 스캔)
- fulltext, index_merge
- index
	- 인덱스 풀 스캔 - 인덱스 블록을 처음부터 끝까지 탐색
	- 인덱스 테이블은 기본 테이블보다 작기 때문에 테이블 풀스캔보다는 효율적
- ALL
	- 테이블 풀 스캔

**possible_keys**
- 옵티마이저가 사용할 수 있는 인덱스 목록

**key**
- 실제로 사용한 인덱스
- 인덱스를 사용하지 않거나 비효율적이라고 생각한다면 튜닝 대상이 될 수 있다.

**ref**
- 조인할 때 어떤 테이블에 엑세스 되었는지 표시

**rows**
- SQL문을 수행할 때 접근한 데이터의 모든 row 수
- EXPLAIN은 MySQL 통계 정보를 가지고 예측한 값, 실제 실행한 뒤의 계획과 다를 수 있다.




## Ref

[MySQL 실행 계획과 결과 컬럼 설명 (MySQL EXPLAIN Output Format)](https://kukim.tistory.com/128)
[](https://velog.io/@bae_mung/TIL-MySQL-Hint)
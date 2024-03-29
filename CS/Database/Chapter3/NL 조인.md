## [[인덱스]]를 이용하는 조인 방식?

일반적으로 NL 조인은 Outer와 Inner 양쪽 테이블 모두 인덱스를 이용한다.
- Outer 테이블의 사이즈가 크지 않다면 Table Full Scan을 사용해도 무방하다. (한 번만 수행하기 때문)
- ==그러나 **Inner 테이블**은 인덱스를 사용해야 한다!== (Outer 루프의 반복 횟수만큼 Table Full Scan)

그렇기 때문에 NL 조인은 인덱스를 이용하는 조인 방식이라고 할 수 있다.
- 소트 머지 조인, 해시 조인도 프로세싱 과정이 NL 조인과 크게 다르지 않다.

## NL 조인 실행계획 제어

NL 조인의 실행계획은 다음과 같다.

![[Pasted image 20231228185038.png|500]]

각 테이블 액세스 시 인덱스를 사용한다는 점도 확인할 수 있었다.

### 힌트를 이용한 튜닝

![[Pasted image 20231228185415.png|400]]

NL 조인을 제어할 때는 `/*+ ordered use_nl(c) */`와 같은 힌트를 사용할 수 있다.

`ordered` 힌트는 FROM 절에 기술한 순서대로 조인하라고 지시할 때 사용한다.
`use_nl` 힌트는 NL 방식으로 조인하라고 지시할 때 사용한다.

즉, 사원 테이블을 기준으로 고객 테이블과 NL 조인하라는 뜻이다.

`leading(e, c)` 힌트를 사용하면 FROM 절을 바꾸지 않고도 마음껏 순서를 제어할 수 있다.

순서에 관한 힌트를 기술하지 않으면 옵티마이저가 스스로 정한다.

## 수행 과정 분석

다음과 같은 SQL문이 있다.

![[Pasted image 20231228190954.png|500]]

1,2는 인덱스 스캔 조건, 3,4는 필터 조건이라고 할 수 있다.

SQL 실행계획은 다음과 같다.

![[Pasted image 20231228191348.png|500]]

고로, 조건절 비교 순서는 2 -> 3 -> 1 -> 4와 같다.

이를 도식화하면 다음과 같다.
- **한 레코드 씩 순차적으로 진행함을 알 수 있다.**
- 이러한 특징 덕분에 부분범위 처리 활용하면 큰 테이블을 조인하더라도 준수한 응답 속도를 낼 수 있다.

![[Pasted image 20231228191250.png|500]]

## 튜닝 포인트

인덱스를 이용하는 조인 방식이므로 [[인덱스 튜닝 기법]]을 적용할 수 있다.

### Outer 테이블의 인덱스 튜닝

Outer 테이블의 인덱스 스캔 단계에서의 일량을 줄인다.
- 전반적으로 영향을 미친다.
테이블 랜덤 액세스 횟수를 최적화할 수 있다.

### Inner 테이블의 인덱스 튜닝

조인 액세스 횟수를 최적화할 수 있다.
- Outer 테이블에서 추출한 레코드의 개수, Inner 테이블의 인덱스 depth 등이 영향을 미친다.
테이블 랜덤 액세스 횟수를 최적화할 수 있다.

## NL 조인 특징 요약

1. 랜덤 액세스 위주의 조인 방식이다.
	1. 인덱스 구성이 아무리 완벽해도 대량 데이터 조인에 불리하다.
2. 한 레코드씩 순차적으로 진행한다.
	1. 부분범위 처리를 활용하면 대량의 데이터라도 매우 빠른 응답속도를 낼 수 있다.
	2. 먼저 액세스되는 테이블의 처리 범위에 의해 전체 일량이 결정된다.
3. 인덱스 구성 전략이 특히 중요하다.

**결론적으로 소량의 데이터를 주로 처리하거나, 부분범위 처리가 가능한 OLTP 시스템에 적합한 조인 방식이다.**
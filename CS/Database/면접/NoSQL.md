관계형이 아닌 데이터 모델을 총칭한다.
- Document 모델
- Key-Value 모델
- Graph 모델
- ...

## 목적

DB의 확장 가능성과 수정 가능성을 염두에 두고 있을 경우 사용한다.

## 특징

##### 다양한 방식으로 데이터를 표현한다.

![[Pasted image 20231220191336.png|500]]
- **Document**: MongoDB
- **Key-Value**: Redis
##### 테이블 사이에 명시된 제약이나 규칙이 딱히 없다.
##### Schema가 고정적이지 않고 매우 유연하다.
##### Scale Out이 쉽다.
- 테이블 사이에 관계가 존재하지 않기 때문이다.
##### 연산이 빠르다.
- 빅데이터, 실시간 연산에 적합하다.


## Ref

[관계형 vs. NoSQL 언제 무엇을 써야할까?](https://ud803.github.io/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4/2021/11/16/RDB-vs.-NoSQL-%EC%96%B8%EC%A0%9C-%EB%88%84%EA%B5%AC%EB%A5%BC-%EC%8D%A8%EC%95%BC%ED%95%A0%EA%B9%8C/)
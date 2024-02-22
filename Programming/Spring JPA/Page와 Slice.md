## 서론

Spring Data JPA에서 pagination query를 만들기 위해서는 다음 조건을 만족해야 한다.
1. 1. `Pageable` 객체를 repository method의 마지막 인수로 전달한다.
2. repository method의 return type을 지정한다.

이 때, repository method의 return type은 크게 3가지이다.
1. `Page`
2. `Slice`
3. `List`

`Page`와 `List`는 익숙한데 `Slice`..? 얘는 도대체 뭘까?


## 결론

무한스크롤 등을 구현할 때는 `Slice`를 사용하는 것이 성능 상 더 유리하다고 한다.
- 한 번 성능 테스트를 해보는 것도 좋을 것 같다!!!

## Ref

[Slice & Page](https://colour-my-memories-blue.tistory.com/10)
[Page<> vs Slice<> when to use which?](https://stackoverflow.com/questions/49918979/page-vs-slice-when-to-use-which)
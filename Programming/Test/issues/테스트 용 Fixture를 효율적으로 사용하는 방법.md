---
sticker: emoji//1f914
---

테스트 용 Fixture를 만들어서 테스트 객체를 생성하고 있다.
그러나 연관관계를 가지고 있는 엔티티들이 존재할 경우 이런 정합성을 맞추기가 힘들다.

DB처럼 데이터 정합성을 지킬 수 있는 Fixture를 만들 수는 없을까?
- 응 안돼~ 직접 넣어주는게 답이야

## 몸으로 배운 철칙

#### 1. 어줍잖게 Auto Increment를 구현하려고 하지 말자

id는 무조건 직접 넣어줘야 한다.

왜냐하면 컨트롤하기 쉽고, 테스트 메서드 내에서도 id가 명확히 보이기 때문에 테스트 코드를 이해하기 쉽다.

무엇보다 repository 테스트를 할 때, `save()` 메서드 호출 시, id가 있으면 이미 DB에 존재한다고 인식한다.

평소에는 문제가 되지 않아 인식하지 못했는데
`save()`할 경우 save하는 객체에 id가 이미 있으면 `persist()`가 아닌 `merge()`로 수행이 되는 것 같다.
- Cascade를 사용할 때 많이 애먹었다. (CascadeType은 `PERSIST`로 설정하였다)

```java
@Transactional  
@Override  
public <S extends T> S save(S entity) {  
  
    Assert.notNull(entity, "Entity must not be null");  
  
    if (entityInformation.isNew(entity)) {  
       entityManager.persist(entity);  
       return entity;  
    } else {  
       return entityManager.merge(entity);  
    }  
}
```

`save()`는 내부적으로 해당 엔티티가 새로운 엔티티인지 확인한다.
만약 엔티티의 id가 **null**(Reference Type)이거나 **0**(Primitive Type)이라면 새로운 엔티티로 보고 `persist()`를 수행한다.
그 외의 경우는 `merge()`를 수행한다.

나의 경우에는 Fixture에 미리 id가 설정되어 있었기 때문에 `merge()`가 호출된 것이다.
CascadeType이 `PERSIST`이니 영속 이외의 다른 연산은 전파되지 않는다.
고로 `merge()`를 수행할 수 없었기 때문에 문제가 된 것이다.
- CascadeType을 `MERGE`로 변경하면 잘 된다!
- ==이 부분은 Cascade를 어떻게 처리하는 지 조사가 필요할 것 같다==

`merge()`는 먼저 DB에 `select` 쿼리를 날린다.
이 때, 레코드가 조회되면 `update` 쿼리를 실행하고 조회되지 않으면 `insert` 쿼리를 실행한다.


#### 2. 연관관계 컬럼은 고정된 Fixture를 사용하지 말자

연관관계가 있는 엔티티 Fixture를 만들 때 귀찮아서 연관관계 컬럼 엔티티를 Fixture로 생성하여 고정해두었다.

```java
// MealLog.java
public static Member member = MemberFixture.DEFAULT.get();    // 머 이런식으루...
```

Fixture에 고정된 값(static)으로 있기 때문에 여러 곳에서 Fixture를 사용해도 그 안의 member는 변하지 않는다.
그런데, 데이터베이스를 사용할 때 이 부분 때문에 정합성이 깨졌다.

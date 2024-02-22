---
sticker: emoji//2705
---
## 발생 이유

JPA에서는 컬렉션을 다른 컬렉션으로 교체해서는 안된다.
이미 있는 컬렉션으로 엔티티가 있는지 없는지, 추가 되었는지 관리하기 때문이다.

Cascade column을 업데이트하는 코드는 다음과 같았다.

```java
public void updateMeals(List<Meal> meals) {  
    this.meals = new ArrayList<>();  
    this.meals.addAll(meals);  
}
```

컬렉션 타입은 영속화 될 때 `org.hibernate.collection.internal.PersistentBag` 타입으로 래핑된다.
이 `PersistentBag`을 이용하여 영속성 전이와 고아 객체를 추적하는데
**새로운 컬렉션을 생성하여 참조를 변경**하면 `PersistentBag` 인스턴스와 엔티티의 참조가 끊어져 문제가 발생한다.

## 해결 방법

새로운 컬렉션을 생성하여 참조를 변경하는 것이 아니라, 기존의 컬렉션을 `remove`, `clear` 등으로 비운 후 다시 엔티티를 추가하면 된다.

```java
public void updateMeals(List<Meal> meals) {  
    this.meals.clear();  
    this.meals.addAll(meals);  
}
```


## Ref

[JPA Update에 관한 질문입니다.](https://www.inflearn.com/questions/211295/jpa-update%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)
[orphanRemoval 옵션 사용시 Collection 참조를 변경하지 말자](https://jerry92k.tistory.com/44)
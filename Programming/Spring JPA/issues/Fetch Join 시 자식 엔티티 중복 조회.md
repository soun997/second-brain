---
sticker: emoji//2705
---
## 문제 상황

부모 엔티티 조회 시 자식 엔티티까지 모두 조회하고자 했다.
부모 엔티티는 `@OneToMany` 연관관계인 자식 엔티티를 두 개 가지고 있기 때문에 fetch join을 두 번 해버리면[[MultipleBagFetchException]] 예외가 발생할 수 있다.
그러므로 자식 엔티티 하나는 [[Fetch Join]]으로, 하나는 batch_fetch_size 설정을 통해 N+1 문제를 방지하였다.

그러나 결과는 예상과 달랐다.
fetch join을 사용한 자식 엔티티가 중복되어 조회되는 것이었다.

Fetch Join을 적용한 부분의 쿼리는 다음과 같다.

```
Hibernate: 
    select
        m1_0.meal_log_id,
        m1_0.created_at,
        m1_0.meal_type,
        m2_0.meal_log_id,
        m2_0.meal_id,
        m2_0.calorie,
        m2_0.carbs,
        m2_0.created_at,
        m2_0.fat,
        m2_0.intake,
        m2_0.meal_data_id,
        m2_0.modified_at,
        m2_0.protein,
        m1_0.member_id,
        m1_0.modified_at 
    from
        meal_log m1_0 
    join
        meal m2_0 
            on m1_0.meal_log_id=m2_0.meal_log_id 
    join
        meal_image m3_0 
            on m1_0.meal_log_id=m3_0.meal_log_id 
    where
        cast(m1_0.created_at as date)=? 
        and m1_0.member_id=?
```

흠... 아직 무엇이 문제인 지 잘 모르겠다.
조회된 자식 엔티티들을 순회하여 id를 조회하였다.

```
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEALS SIZE]: 12
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 1
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 1
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 1
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 1
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 2
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 2
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 2
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL ID]: 2
...
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL_IMAGE ID]: 1
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL_IMAGE ID]: 2
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL_IMAGE ID]: 3
2024-01-03T08:38:07.080+09:00  INFO 27788 --- [nio-8080-exec-4] c.k.v.meallog.service.dto.MealLogDto     : [MEAL_IMAGE ID]: 4
```

자식 엔티티 A, B가 있다면 (A의 개수 X B의 개수) 만큼 조회가 되어버렸다.
(이게 그 유명한 카데시안 곱인가..?)

내가 작성한 JPQL 쿼리는 다음과 같다.

```java
@Query(  
        " select m from MealLog m join fetch m.meals join m.mealImages"  
                + " where cast(m.createdAt as localdate) = :date and m.member.id = :memberId")  
List<MealLog> findAllByDate(LocalDate date, Long memberId);
```


## 발생 이유

근본적인 문제가 무엇인지에 대해 접근하고자 했다.
왜 카테시안 곱이 발생하는 것일까?

일단 엔티티들의 구조는 다음과 같다.
- 부모 엔티티: `MealLog`
- 자식 엔티티1: `Meal` -> 하나의 `MealLog` 당 3개
- 자식 엔티티2: `MealImage` -> 하나의 `MealLog` 당 4개

1. 먼저 fetch join으로 `MealLog`를 불러오면서 `Meal`을 함께 불러온다.
	1. **이 때 총 3개의 `MealLog` + `Meal` 레코드를 조회한다.
2. 3개의 레코드에 4개의 `MealImage`를 조인한다.
	2. **최종적으로 총 12개의 레코드를 조회한다.**

데이터베이스 상에서는 12개의 레코드를 조회하는 것이 맞다.
그러나 우리는 이를 하나의 `MealLog` 엔티티에 매핑하고자 한다.
그러므로 **중복을 제거**해야 한다.


## 해결 방법

JPQL에 `distinct` 키워드를 붙이면 해결된다.

```java
@Query(  
        " select distinct m from MealLog m join fetch m.meals join m.mealImages"  
                + " where cast(m.createdAt as localdate) = :date and m.member.id = :memberId")  
List<MealLog> findAllByDate(LocalDate date, Long memberId);
```

JPQL에서 사용하는 `distinct`는 데이터베이스의 조회(SQL) 결과에는 적용되지 않는다.
그러나 이를 객체와 매핑하는 과정에서 위에서 발생하는 중복을 제거해준다.
이렇게 N+1 문제도 해결하고 fetch join을 사용하여 쿼리를 한 개 더 줄일 수 있었다.

어찌보면 당연한 결과였을텐데 JPA와 SQL에 대한 이해도가 떨어져서 발생한 이슈였던 것 같다.


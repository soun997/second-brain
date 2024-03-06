## 서론

Fetch Join을 사용하면 `limit` 쿼리를 통한 Pagination을 사용할 수 없다는 블로그 글을 보게 되었다.
- [JPA에서 Fetch Join과 Pagination을 함께 사용할때 주의하자](https://tecoble.techcourse.co.kr/post/2020-10-21-jpa-fetch-join-paging/)

생각해보니 프로젝트 코드에서도 Fetch Join과 Pagination을 함께 사용하는 부분이 있어 빠르게 실험해보았따.


## 진짜 안되나?

### JPA 로그

```sql
Hibernate: 
    select
        distinct m1_0.meal_log_id,
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
    where
        cast(m1_0.created_at as date)=? 
        and m1_0.member_id=?
```

진짜 `limit` 쿼리가 나타나지 않은 것을 확인할 수 있었다.

```
HHH90003004: firstResult/maxResults specified with collection fetch; applying in memory
```

덤으로 memory에서 Pagination을 적용하여 반환했다는 경고 메세지도 날려주었다.


## Ref

[Fetch join과 Paging](https://lsj31404.tistory.com/entry/Fetch-join%EA%B3%BC-Paging)
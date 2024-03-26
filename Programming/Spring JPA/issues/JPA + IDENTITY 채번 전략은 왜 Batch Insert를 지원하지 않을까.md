## Hibernate의 Batch Insert 비활성화

**`IDENTITY` 채번 전략**을 사용할 경우, Hibernate는 JDBC 수준에서 Batch Insert를 비활성화 한다.
- 새로 할당할 Key 값을 미리 알 수 없기 때문에, Hibernate의 flush 방식인 'Transaction Write Behind'와 충돌이 발생하게 된다.

### Transaction Write Behind (쓰기 지연)

영속성 컨텍스트에 변경이 발생했을 때, 바로 데이터베이스로 쿼리를 보내지 않고
SQL 쿼리를 버퍼에 모아놨다가 `flush()`하는 시점에 데이터베이스로 보낸다.
- `flush()`가 발생하는 경우
	1. 직접 호출
	2. JPQL 실행
	3. 트랜잭션 `commit()`

### 무슨 충돌이 발생한다는 거야?

```java
@Entity
public class Parent {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  

	private String name;
  
    @OneToMany(  
            mappedBy = "parent", 
            fetch = FetchType.LAZY,  
            cascade = CascadeType.ALL)  
    private List<Child> children;  
}

@Entity
public class Child {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  

	private String name;
  
    @ManyToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "parent_id")  
    private Parent parent;
}
```

간단한 엔티티들을 작성해봤다.
Parent는 Child와 `OneToMany` 관계를 가지고 있다.

`IDENTITY` 전략의 경우, DB에 insert 될 때, 비로소 고유한 `id`를 가지게 된다.
그러므로로 **실제 DB에 부모 엔티티를 insert**한 뒤, **반환된 `id`를 FK로 자식 엔티티를 insert** 해야 한다.

만약 부모 엔티티를 insert하는 쿼리를 모아서 실행한다면?
나중에 자식 엔티티를 insert할 때, **어떤 부모 엔티티에 매핑해야 하는지 알기 어려워진다.**

```sql
INSERT INTO parent(name)
VALUES 
	("부모1"),
	("부모2"),
	("부모3");
  
INSERT INTO child(name, parent_id)
VALUES
	("자식1", ?),
	("자식2", ?),
	("자식3", ?),
	("자식4", ?),
	("자식5", ?);  // 누굴 누구에게 매핑해야 하지?
```


## `SEQUENCE`와의 비교




## Ref

[Batch inserts](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch-insert)
[JPA의 쓰기지연 기능 확인해보기 (transactional write-behind)](https://soongjamm.tistory.com/150)
https://thorben-janssen.com/jpa-generate-primary-keys/#GenerationTypeIDENTITY
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
@NoArgsConstructor
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

	// ...
}

@Entity
@NoArgsConstructor
public class Child {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  

	private String name;
  
    @ManyToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "parent_id")  
    private Parent parent;

	// ...
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

엔티티의 채번 전략을 `SEQUENCE`로 수정한다.

```java
@Entity  
@NoArgsConstructor  
@SequenceGenerator(  
        name = "PARENT_SEQ_GENERATOR",  
        sequenceName = "PARENT_SEQ",  
        initialValue = 1, allocationSize = 1  
)  
public class Parent {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "PARENT_SEQ_GENERATOR")  
    private Long id;  
    private String name;  
  
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)  
    private List<Child> children = new ArrayList<>();  
}

@Entity  
@NoArgsConstructor  
@SequenceGenerator(  
        name = "CHILD_SEQ_GENERATOR",  
        sequenceName = "CHILD_SEQ",  
        initialValue = 1, allocationSize = 1  
)  
public class Child {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "CHILD_SEQ_GENERATOR")  
    private Long id;  
  
    private String name;  
  
    @ManyToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "parent_id")  
    private Parent parent;  
}
```

### Batch Insert 실행 전 채번

애플리케이션을 실행하면 다음과 같이 `SEQUENCE` 객체가 생성되었다는 로그를 출력한다.

```sql
Hibernate: 
    create sequence child_seq start with 1 increment by 1
Hibernate: 
    create sequence parent_seq start with 1 increment by 1
```

Batch Insert 이전에 먼저 채번한다.

```sql
Hibernate: 
    select
        next value for parent_seq
Hibernate: 
    select
        next value for child_seq
Hibernate: 
    select
        next value for child_seq
Hibernate: 
    select
        next value for child_seq
Hibernate: 
    select
        next value for child_seq
Hibernate: 
    select
        next value for child_seq

```

#### Batch 채번을 통한 최적화

현재 `@SeqenceGenerator`의 `allocationSize`를 1로 설정했기 때문에 **삽입하려는 엔티티의 개수만큼 채번 쿼리가 발생**했다.

`allocationSize`를 더 크게 조정하여, 이들을 한 번에 가져올 수 있다. (default는 50이다)

```java
@Entity  
@NoArgsConstructor  
@SequenceGenerator(  
        name = "PARENT_SEQ_GENERATOR",  
        sequenceName = "PARENT_SEQ",  
        initialValue = 1, allocationSize = 50  
)  
public class Parent {
```

### Batch Insert 쿼리

```sql
INSERT INTO parent(id, name)
VALUES 
	(1, "부모1")
  
INSERT INTO child(name, parent_id)
VALUES
	("자식1", 1),
	("자식2", 1),
	("자식3", 1),
	("자식4", 1),
	("자식5", 1);
```




## Ref

[Batch inserts](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch-insert)
[JPA의 쓰기지연 기능 확인해보기 (transactional write-behind)](https://soongjamm.tistory.com/150)
https://thorben-janssen.com/jpa-generate-primary-keys/#GenerationTypeIDENTITY
https://lordofkangs.tistory.com/358
https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/
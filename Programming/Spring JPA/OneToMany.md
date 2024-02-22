1:N 관계에서는 1이 연관관계의 주인이다.
- 1인 쪽에서 외래키를 관리하겠다는 의미가 된다.

**표준스펙에서 지원은 하지만 실무에서 사용하는 것을 권장하지는 않는다!**

![[Pasted image 20231218011116.png|500]]

Team은 Member를 가지지만, Member는 Team을 가지지 않는다.
객체 입장에서는 이 편이 더 자연스러운 설계일 것이다.

그러나 DB 입장에서는 무조건 일대다의 '다' 쪽에 FK가 들어간다.
Team에서 Member가 바뀌면 DB의 Member 테이블에 업데이트 쿼리가 나가게 된다.


## 예제 코드

```java
@Entity
public class Article {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String content;

	// 아무것도 적지 않으면 @JoinTable로 동작한다.
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true) 
    private List<Image> images = new ArrayList<>();

    public void addImage(final Image image) {
        images.add(image);
    }
}
```

```java
@Entity
public class Image {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String url;
}
```



## 일대다 단방향 매핑

`@OneToMany`와 `@JoinColumn` 혹은 `@JoinTable` 어노테이션을 사용해서 매핑해줄 수 있다.

### `@JoinTable` 사용

#### 데이터 저장

```
Hibernate: insert into article (id, content) values (null, ?)

Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)

Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
```

Article과 Image를 저장한 후, 매핑 테이블에 한 번 더 저장해 준다.
이는 다대다 관계가 되어버리며, 매우 효율적이지 않다.

#### 데이터 삭제

Image를 삭제하고자 한다.
일반적인 방법처럼 `imageRepository`를 이용해서 삭제하면, 매핑 테이블에 아직 데이터가 존재하기 때문에 FK 제약조건으로 삭제가 불가능하다.

Article에 있는 Image 리스트에서 해당 Image를 삭제해주면 된다.

```
Hibernate: delete from article_images where article_id=?
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: insert into article_images (article_id, images_id) values (?, ?)
Hibernate: delete from image where id=?
```

article_images에서 1번, image에서 1번 삭제될 것이라 예상했지만 결과는 달랐다.
1. article_image에서 article_id를 통해 모두 지운다.
2. 지우려는 image를 제외한 나머지 image를 article_images에 다시 저장한다.
3. 지우려는 image를 테이블에서 삭제한다.

이 얼마나 비효율적인가!


### `@JoinColumn` 사용

```java
public class Article{
    ....
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name="article_id")
    private List<Image> images = new ArrayList<>();

    ...
```

위와 같이 `@JoinColumn` 어노테이션을 통해 명시해주어야 한다.

#### 데이터 저장

```
Hibernate: insert into article (id, content) values (null, ?)

Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)
Hibernate: insert into image (id, url) values (null, ?)

Hibernate: update image set article_id=? where id=?
Hibernate: update image set article_id=? where id=?
Hibernate: update image set article_id=? where id=?
Hibernate: update image set article_id=? where id=?
```

`@JoinTable`처럼 새로운 매핑 테이블이 생기지는 않지만, image를 insert하는 횟수만큼 update 쿼리가 함께 실행된다.
- 일단 article_id가 없는 상태로 image를 저장하고, 후의 update 쿼리로 article_id를 설정해준다.

#### 데이터 삭제

```
Hibernate: update image set article_id=null where article_id=? and id=?
Hibernate: delete from image where id=?
```

image의 article_id를 null로 초기화시키고, 해당 image를 삭제한다.


## 일대다 양방향 매핑

```java
@Entity
public class Article{

    @OneToMany(mappedBy = "article",cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Image> images = new ArrayList<>();

    public void addImage(final Image image) {
        images.add(image);
        image.setArticle(this);
    }

    public void removeImage(final Image image){
        images.remove(image);
        image.setArticle(null);
    }
}

@Entity
public class Image {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "article_id")
    private Article article;

      ...
}
```

FK를 가지고 있는 쪽인 Image에 `@ManyToOne`과 `@JoinColumn` 을 설정해주면 된다.
- 상대 엔티티 테이블에 FK 컬럼이 있음을 알려준다.
`@OneToMany` 쪽에는 `mappedBy` 속성을 넣어주어 주인임을 명시합니다.

#### 데이터 저장

```
Hibernate: insert into article (id, content) values (null, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
Hibernate: insert into image (id, article_id, url) values (null, ?, ?)
```

#### 데이터 삭제

```
Hibernate: delete from image where id=?
```

내가 원하는 대로 쿼리가 딱 한 번씩만 실행되는 것을 확인했다!


## Ref

[JPA - One To Many 단방향의 문제점](https://dublin-java.tistory.com/51### )
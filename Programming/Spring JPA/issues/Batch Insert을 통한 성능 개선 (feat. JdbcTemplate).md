## 😢문제 상황

플로깅 기록 서비스인 '줍줍'에서는 사용자의 이동 경로(`Route`)를 30초 마다 수집하고, 플로깅 종료시 이를 한꺼번에 서버에 전송한다.
사용자가 약 1시간 동안 플로깅을 하면 120개의 위치 정보 데이터가 수집된다.

### 무엇이 문제인가?

Entity의 `@GeneratedValue` 전략으로 `IDENTITY`를 사용하고 있기 때문에 JPA의 `쓰기 지연`을 사용할 수 없다.
- [[JPA + IDENTITY 채번 전략은 왜 Batch Insert를 지원하지 않을까]]
즉, **단일 Insert 쿼리가 120번 발생**하게 되는 것이다.

부하 테스트를 진행해서 현재의 로직이 성능 목표치를 만족하는지 확인하겠다.
- 1000VU, 3분 동안 부하테스트

![[Pasted image 20240326104724.png]]
#### 테스트 결과
- `p95 <= 500ms`: 달성 실패 ❌
- `p99 <= 1s`: 달성 실패 ❌

**대량으로 발생하는 단일 Insert**를 성능 저하의 원인으로 규정했다.
- [[단일 Insert가 문제가 되는 이유]]

### 목표

현재 사용하고 있는 DB인 MySQL은 `SEQUENCE` 전략을 지원하지 않는다.
Hibernate가 `SEQUENCE`를 모방하기 위한 sequence table을 생성해주긴 하지만...
Insert 전에 아래와 같이 기본키를 생성해주는 쿼리가 추가로 발생한다.
- 동시성 보장을 위해 락도 걸린다.

```sql
Hibernate
select
	next_val as id_val
from
	hibernate_sequence for update

Hibernate:
update
	hibernate_sequence
set
	next_val= ?
where
	next_val=?
```

성능 저하가 우려되므로 `SEQUENCE` 전략을 사용하기는 어려울 것 같다.

그러므로 **`IDENTITY` 전략을 유지하면서, 일괄 Insert가 가능하도록 개선하고자 한다.**


## 🤔해결 과정

### JdbcTemplate 적용

```java
public void saveBatch(Route route) {  
    KeyHolder keyHolder = new GeneratedKeyHolder();  
    jdbcTemplate.update(new PreparedStatementCreator() {  
        @Override  
        public PreparedStatement createPreparedStatement(Connection conn) throws SQLException {  
            PreparedStatement pstmt = conn.prepareStatement(  
                    "INSERT INTO ROUTE(`plogging_log_id`) VALUE (?)", new String[] {"id"});  
            pstmt.setLong(1, route.getPloggingLog().getId());  
            return pstmt;  
        }  
    }, keyHolder);  
    batchInsert(Objects.requireNonNull(keyHolder.getKey()).longValue(), route.getLocations());  
}

private void batchInsert(Long routeId, List<Location> locations) {  
    jdbcTemplate.batchUpdate(  
            "INSERT INTO LOCATION (`latitude`, `longitude`, `route_id`) VALUES (?, ?, ?)",  
            new BatchPreparedStatementSetter() {  
                @Override  
                public void setValues(PreparedStatement ps, int i) throws SQLException{  
                    ps.setBigDecimal(1, locations.get(i).getLatitude());  
                    ps.setBigDecimal(2, locations.get(i).getLatitude());  
                    ps.setLong(3, routeId);  
                }  
                @Override
                public int getBatchSize() {  
                    return locations.size();  
                }  
            });  
}
```







## 😎결과

![[Pasted image 20240326104827.png]]



## 🏃다음 단계

[[NoSQL 전환을 통한 성능 개선]]


## 📚Ref

https://dkswnkk.tistory.com/682
https://datamoney.tistory.com/319
https://jojoldu.tistory.com/558
https://cheese10yun.github.io/jpa-batch-insert/
https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/
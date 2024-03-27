## 쓰기 지연 전략

Hibernate는 기본적으로 **쓰기 지연 전략(Transaction Write Behind)** 을 사용한다.
쓰기 지연 전략이란 `insert`, `update` 등의 데이터 변경 작업을 즉시 처리하지 않고 모아놨다가, `flush()`할 때 한 번에 DB에 기록하는 방식이다.
> IDENTITY 채번 전략을 사용하는 경우, 쓰기 지연 전략을 사용하지 않는다.
> [[JPA + IDENTITY 채번 전략은 왜 Batch Insert를 지원하지 않을까]]

처음엔 이 쓰기 지연 전략이 Multi-Row Insert와 동일한 것인 줄 알았다.
그러나 DB에 기록된 쿼리 로그를 확인하면 


## 영향을 주는 속성

`spring.jpa.properties.hibernate.jdbc.batch_size`
`spring.jpa.properties.hibernate.order_inserts`
`spring.jpa.properties.hibernate.order_updates`

## Batch Size 설정 X

```
Session Metrics {
    831600 nanoseconds spent acquiring 1 JDBC connections;
    0 nanoseconds spent releasing 0 JDBC connections;
    1928800 nanoseconds spent preparing 22 JDBC statements;
    19491500 nanoseconds spent executing 22 JDBC statements;
    0 nanoseconds spent executing 0 JDBC batches;
    0 nanoseconds spent performing 0 L2C puts;
    0 nanoseconds spent performing 0 L2C hits;
    0 nanoseconds spent performing 0 L2C misses;
    60499400 nanoseconds spent executing 1 flushes (flushing a total of 18 entities and 3 collections);
    0 nanoseconds spent executing 0 partial-flushes (flushing a total of 0 entities and 0 collections)
}
```

## `application.yml` 설정

### MySQL
```yaml
datasource:  
  driver-class-name: org.mariadb.jdbc.Driver  
  url: jdbc:mariadb://localhost:3306/test?rewriteBatchedStatements=true  
```

### Postgresql
```yaml
datasource:  
  driver-class-name: org.postgresql.Driver 
  url: jdbc:mariadb://localhost:3306/test?reWriteBatchedInserts=true
```


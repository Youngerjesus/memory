# infra 공방 3주차

## 데이터 조회 개선

대부분 웹 어플리케이션은 데이터를 조회하고 저장하는 작업이 주를 이룬다.

즉 SQL 과 관련된 작업이 70% 이상 이라서 이 부분을 최적화 할 필요가 있다.

## 서브쿼리 사용

- 서브 쿼리는 총 세군데에 쓸 수 있다. FROM, SELECT, WHERE
- 여러 쿼리들을 조합하고 싶을 때 서브 쿼리를 사용하고 테이블을 결합하고 싶을 때 조인을 쓴다.

## MySQL 최적화 대상

### Clients

- 쿼리 요청 수 자체를 줄이는 방법.
- Connection Pool 사용
- Paging 으로 요청 사이즈 줄이기

## ETC

- Query Caching 은 MySQL 8.0 부터 제거되었다.
- MySQL 5.7 이상은 InnoDB 를 이용하고 여기선 B-Tree 인덱스를 기본적으로 사용한다.

## 인덱스를 사용할 수 없는 조건들

- 인덱스 칼럼을 가공한 경우.
- 자동 형변환이 적용된 경우 (가공된 경우와 결과적으로 같음)
- LIKE %문자열% 이런 식으로 검색을 하는 경우.
- Where 절에 OR 연산이 있는 경우.
- Where 절에 IN 연산이 있는 경우.
- 인덱스가 걸려 있어도 정렬된 순서를 고려하지 못하면 인덱스를 사용하지 않을 수 있다.

## 인덱스를 통한 성능 향상

- 인덱스 스캔을 효율화
    - 수직적 스캔과 수평적 스캔을 효율적으로 탐색하는 것.
    - 인덱스 스캔을 할 때 풀 스캔을 하지 않도록 해야한다. ⇒ 첫 시작 지점을 찾을 수 있어야 한다. or 시작 지점을 여러개로 만들지 않는다.
    - 인덱스 칼럼이 여러개로 이뤄저있다면 범위 조건의 검색은 되도록 뒤에서 쓰는 것. 즉 인덱스 선행 칼럼이 모두 ‘-’ 일 때.
    - 범위 조건이 몇 개 없다면 between 대신에 IN-List 를 써서 Index Skip Scan 으로 푸는 것도 괜찮아 보인다.
    - Between 과 LIKE 둘 다 사용이 가능하다면 Between 을 쓰는 것. Between 이 범위가 더 적다.
- 인덱스 스캔 후 랜덤 엑세스를 최소화
    - 테이블 엑세스를 했는데 허빵을 치지 않도록 하는 것을 말한다.
    - 인덱스 클러스터링 팩터를 높여서 데이터가 모여있도록 하는 것. 이를 통해서 테이블 엑세스를 줄이는 것.
    - 인덱스에 칼럼을 추가해서 충분한 정보를 주도록 해서 허빵을 치는 걸 막는 방법.
        - 이 방법은 새로운 인덱스를 추가하는 방법도 있지만 기존의 인덱스에 추가하는 방법도 있다. 새로운 인덱스를 추가하면 DML 부하를 준다. (웬만하면 기존 인덱스에 추가하는 방법이 나은듯)
        - 인덱스를 대체하는 방법은 추천하지 않는다. 기존에 잘 쓰고 있는 인덱스가 있으니.
    - 커버링 인덱스로 인덱스만 읽고 처리하도록 하는 것.
- 인덱스를 사용할 경우 해당 칼럼으로 정렬이 되어 있다. 이를 이용하는 것.
- 테이블 엑세스를 안하도록 클러스터링 인덱스를 사용하는 것. 테이블 자체를 인덱스 구조로.
    - MySQL 은 테이블 당 한 개만 생성이 된다. Primary Key.

## 조인을 통한 성능 향상

- NL 조인에서 인덱스를 타도록 하는 것. (특히 Inner 테이블에서)
- 인덱스 스캔 + 랜덤 엑세스의 비용을 생각해서 Outer 테이블과 Inner 테이블을 선택하자.
- Outer 테이블의 크기를 줄여서 인덱스 횟수를 줄이는 방법도 있다.

## 인덱스를 사용해도 성능이 안나오는 사례들

- 소량의 데이터만 검색하지 않은 경우 (인덱스 검색을 통한 I/O 는 Multi I/O 가 아님) 그리고 대량의 데이터를 검색하는 경우 버퍼 풀에서 찾을 확률이 낮아진다. 즉 히트율이 낮아진다.

## 목적에 맞는 적합한 데이터베이스 찾기

- MongoDB, MySQL, HDFS, Cassandra, Elasticsearch
- MySQL 은 대량의 데이터를 검색하는 경우와 파티셔닝이 많이 필요한 경우에는 그렇게 효과적이지 않다.

## 튜닝 절차

1) 일단 조회를 해본다. 결과가 잘 나오는지 확인해보고 목표치 (fetch/duration time) 에 맞는지 확인해본다.

2) 목표치에 맞지 않다면 개선 대상을 파악한다. 개선 대상을 파악할 땐 실행 계획을 보고 개선할 부분을 찾는다. 주로 조인, 서브쿼리 구조, 인덱스, 정렬 등을 파악하고 이 부분을 개선한다.

## 실행계획 읽기

[https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key_len](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain_key_len)

- id: SELECT 에 붙은 순서를 말한다.
- table: 어떤 테이블에 접근하고 있는지 표시
- partitioning: 파티셔닝 되어 있는 경우에 사용되는 필드.
- select_type: SELECT 문의 유형을 의미한다.
    - SIMPLE: 단순 SELECT 문을 의미.
    - PRIMARY: 서브 쿼리를 감싸는 외부 쿼리. UNION 이 포함될 경우 첫번째 SELECT 문
    - SUBQUERy: 독립적으로 수행되는 서브쿼리
    - DERIVED: FROM 절에 작성된 서브쿼리
    - UNION: UNION, UNION ALL 로 합쳐진 SELECT
- type: 테이블에서 어떻게 데이터를 가져올 것인가를 나타내는 하옴ㄱ
    - system: 테이블에 데이터가 없거나 한 개만 있는 경우
    - const: 조회되는 데이터가 단 1건일 때
    - eq_ref: 조인이 수행될 때 드리븐 테이블의 데이터에 PK 혹은 고유 인덱스로 단 1건의 데이터를 조회할 때
    - ref: eq_ref 와 같으나 데이터가 2 건 이상인 경우
    - index: 인덱스 풀 스캔
    - range: 인덱스 레인지 스캔
    - all: 테이블 풀 스캔
- key
    - 옵티마이저가 실제로 선택한 인덱스
- key_len
    - MySQL 이 사용하고 있는 키의 길이로 주로 Multiple-part index 에서 얼마나 많은 칼럼을 인덱스로 사용하고 있는지 알려준다.
- rows
    - SQL 문을 수행하기 위해 접근하는 데이터의 모든 행 수
- ref: 인덱스와 비교될 칼럼
- filtered: 행 데이터를 가져와서 WHERE 절 등으로 필터를 수행하면 전체 몇 프로의 행이 남는지를 표시한다.
- extra
    - Distinct: 중복 제거할 때
    - Using where: WHERE 절로 필터시
    - Using temporary: 데이터의 중간결과를 저장하고자 임시 테이블 생성. 보통 DISTINCT, GROUP  BY, ORDER BY 구문이 포함된 경우 임시 테이블 생성
    - Using Index: 인덱스만 읽어서 처리할 경우
    - Using filesort: 정렬 시
    - Using join buffer: 조인에 적절한 인덱스가 없어서 조인 버퍼를 이용했음을 표시하는 경우.

## DB 통계 정보 사용하기

슬로우 쿼리 확인하기

```sql
SELECT query, exec_count, sys.format_time(avg_latency) AS "avg latency", rows_sent_avg, rows_examined_avg, last_seen
FROM sys.x$statement_analysis
ORDER BY avg_latency DESC;
```

성능 개선 대상 식별

```sql
SELECT DIGEST_TEXT AS query,
             IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
             COUNT_STAR AS exec_count,
             SUM_ERRORS AS err_count,
             SUM_WARNINGS AS warn_count,
             SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS exec_time_total, 
             SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS exec_time_max, 
             SEC_TO_TIME(AVG_TIMER_WAIT/1000000000000) AS exec_time_avg_ms, SUM_ROWS_SENT AS rows_sent,
             ROUND(SUM_ROWS_SENT / COUNT_STAR) AS rows_sent_avg, SUM_ROWS_EXAMINED AS rows_scanned,
             DIGEST AS digest
   FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC
```

인덱스 사용량 및 정보 보기

```sql
SHOW STATUS LIKE '%key%';
```

I/O 요청이 많은 테이블 목록

```sql
SELECT * FROM sys.io_global_by_file_by_bytes WHERE file LIKE '%ibd';
```

테이블별 작업량 통계

```sql
SELECT table_schema, table_name, rows_fetched, rows_inserted, rows_updated, rows_deleted, io_read, io_write
FROM sys.schema_table_statistics
WHERE table_schema NOT IN ('mysql', 'performance_schema', 'sys');
```

최근 실행된 쿼리 이력 기능 활성화

```sql
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history'
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history_long'
```

최근 실행된 쿼리 이력 확인

```sql
SELECT * FROM performance_schema.events_statements_history
```

## DB 서버 튜닝

- thread 개수 조절하기
- innodb_buffer_pool_size 크게 설정하기 (버퍼 풀로 데이터와 인덱스 블록이 저장되는 공간)
- key_buffer: 인덱스를 위한 버퍼 크기로 대용량 데이터를 조회하는 경우가 많다면 이 값을 조절해보자. key_reads 가 많다면 key_buffer_size 가 작아서 일수도 있다.
- max_connections: 서버가 허용하는 최대한의 커넥션 수로 서버의 사양에 따라서 달라질 수 있다. 일반적으로는 120~250 개로 설정하고 고용량의 서버의 경우에는 1000 개 정도로 설정하는 경우도 있다.
- back_log: max_connection 설정값 이상으로 접속이 발생할 경우 얼마만큼의 커넥션을 큐에 보관할지에 대한 설정 값이다.
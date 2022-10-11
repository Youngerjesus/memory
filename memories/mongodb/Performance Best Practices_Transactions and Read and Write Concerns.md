# Performance Best Practices: Transactions and Read / Write Concerns

https://www.mongodb.com/blog/post/performance-best-practices-transactions-and-read--write-concerns

***

## Best Practices for Multi-Document Transactions

- 긴 시간의 트랜잭션을 만들거나, 하나의 트랜잭션에서 너무 많은 연산을 하는 건 좋지 않다.
    - 긴 시간은 여기서 60초를 말함
    - (60초를 주기로 데이터를 계속해서 디스크에 플러쉬 하는데 이거랑도 연관이 있을듯)
    - 어디에 좋지 않냐면 WiredTiger Storage Engine 에 있는 cache 에 좋지 않음.
    - 캐시는 스냅샷 이후의 모든 쓰기 연산 상태들을 가지고 있다.
    - 트랜잭션 동안에는 같은 스냅샷을 쓰고 있지만 새로운 쓰기들은 캐시에 계속해서 누적된다.
    - 이러한 쓰기들은 트랜잭션이 중단되거나 커밋될 때까지 디스크에 플러쉬 할 수 없다.

즉 다음과 같은 룰들을 따라야한다.

### Transaction runtime limit

- multi document transaction 이 60초 지나면 자동으로 버려진다.
    - 60초 안에 해결할려고 트랜잭션을 쪼개야한다.
    - 즉 제한 시간동안에 트랜잭션을 처리하도록 인덱스라던지, 쿼리 패턴이라던지 이를 최적화 해야한다.

### Number of operations in a transaction

- 1000 개 다큐먼트 이상을 한번에 바꾸지마라. 최대 1000개.

### Distributed, multi-shard transactions

- 여러 샤드에 걸쳐서 생기는 트랜잭션은 여러 노드에 걸쳐서 조정되므로 더 큰 비용을 초래한다.
- 멀티 도큐먼트 트랜잭션 중 멀티 샤드에서 일관성 있게 읽을려면 read concern 을 snapshot 으로 지정하면 된다.
    - 레이턴시가 너무 크리티컬 하다면 local 로 설정하자.

### Exception Handling

- 트랜잭션이 실패하면 catch 문에서 잡아서 retry 로직을 넣자.

### Benefit for write latency

- write 연산의 경우에 트랜잭션이 latency 를 줄여줄 수도 있다.
    - 예로 10개의 문서 업데이트를 개별로 하고 `writeConcern` 으로 `w: majority` 걸면 복제 셋에서 연산해야하는 비용이 트랜잭션보다 더 클 수 있다.

## Choose the Appropriate Write Guarantees

- 영속성 보장을 위해서 writeConcern 을 어느정도 수준까지 적용할건지 고민해봐야한다.
    - primary
    - Replication
    - Journal

# MongoDB Transaction

https://www.mongodb.com/docs/manual/core/transactions/#std-label-transactions-write-concern

***

## Transactions API

- new callback API 에서는 `TransientTransactionError` 와 `UnknownTransactionCommitResult` 에 대한 retry 로직이 들어가있다.
- MongoDB 4.2 버전에서 트랜잭션을 이용하고 있다면 드라이버도 이 버전이상이어야한다.
    - 4.2 이전에서는 트랜잭션에서 컬렉션을 만드는게 불가능했음.
    - 4.4 부터는 컬렉션과 인덱슬르 트랜잭션에서 만드는게 가능함.
- 트랜잭션을 이용할려면 session 을 사용해야한다.

## Transactions and Atomicity

- `distributed transactions` 과 `Multi-Document Transactions` 는 동일한 말이다.
- 4.2 부터 shard cluster 에서 트랜잭션이 가능해진다.
- 트랜잭션이 진행중이더라도 읽기는 가능하다.
- `distributed transaction` 의 비용은 비싸다. 하나의 문서 업데이트에 비하면. 그러르모 [비정규화](https://www.mongodb.com/docs/manual/core/data-model-design/#std-label-data-modeling-embedding) 도 고려해볼만하다.

## Transactions and Sessions

- transaction 을 시작할려면 session 을 열어야 하고 세션 하나에 트랜잭션 하나를 열 수 있다.
- 세션이 종료되었는데 트랜잭션이 열려있다면 트랜잭션은 종료된다.

## Read Concern/Write Concern/Read Preference

### Transactions and Read Preference

- `Read Preference` 의 정의는 replica set 의 멤버 중에서 누구한테 데이터를 읽을 건를 말한다.
    - 기본적으로 Primary 에서 읽는다.
    - 나머지 옵션으로는 `primaryPreferred`, `secondary`, `secondaryPreferred`, `nearest` 등이 있다.
- 트랜잭션에서 기본은 `primary` 임.

### Transactions and Read Concern

- `Read Concern` 은 읽을 때 일관성 있게 데이터를 읽어오기 위한 설정이다.
    - 예로 대부분의 노드가 가지고 있는 데이터를 읽어오도록 하는.
    - 기본은 `local` 이다. 이는 롤백될 수 있음.
    - `snapshot` 이라는 것도 있는데 이건 5.0 부터 가능하다. 트랜잭션에서도 설정하도록 가능하고 특징은 다음과 같다.
        - `causally consistent session` 의 경우
            - `write concern: majority`를 따른다.
        - `causally consistent session` 이 아닌 경우
            - 트랜잭션 시작 전에 `write concern: majority` 를 따른다.
        - `causally consistent session` 은 세션이 시작되고 연산이 진행된 경우를 말한다.
        - `causally consistent session` 은 클라이언트 입장에서 데이터를 보내고 받을 때 하나의 세션을 쓰는데 거기서 일관성을 맞추는 경우를 말한다.
            - 예로 이전 연산이 문서를 다 지우고 다음 연산이 결과를 읽는 거라면 결과가 없어야 결과적 일관성어 맞춰지는 것이다.
            - 몽고 DB 3.6 부터 이 기능이 지원되었고 Read, Write 모두 `majority` 여야한다. 그리고 어플리케이션에서 하나의 스레드가 이 세션속에서 명령을 보낸다.

#### Causal Consistent Session Example

```java
ClientSession session1 = client.startSession(ClientSessionOptions.builder().causallyConsistent(true).build());
Date currentDate = new Date();
MongoCollection<Document> items = client.getDatabase("test")
        .withReadConcern(ReadConcern.MAJORITY)
        .withWriteConcern(WriteConcern.MAJORITY.withWTimeout(1000, TimeUnit.MILLISECONDS))
        .getCollection("test");
items.updateOne(session1, eq("sku", "111"), set("end", currentDate));
Document document = new Document("sku", "nuts-111")
        .append("name", "Pecans")
        .append("start", currentDate);
items.insertOne(session1, document);
```

- `client.startSession` 으로 열어야하네.


- 트랜잭션에서 기본은 `session-level read concern` 이다.
    - 기본적으로 `local` 이다.
- 트랜잭션에서는 트랜잭션에서 설정란 `read concern` 이 적용된다. 데이터베이스, 컬렉션 수준의 `read concern` 이 아니라.

### Transactions and Write Concern

- 트랜잭션은 트랜잭션 레벨의 `write concern` 을 사용한다.
- 트랜잭션 안에서 개별적인 쓰기 연산에 `write concern` 을 건다면 에러가난다.
- 기본 값은 버전에 따라 다른데 4.4 이전에는 `w: 1` 이고 5.0 이후부터는 `w: majority` 이다.
- **샤드 클러스터 트랜잭션의 경우 `w: majority, j: true` 옵션이 적용된다. 트랜잭션 커밋시에 `write concern` 상관없이.**
    - 이렇게 안하면 트랜잭션 쓸 수 없다.
- `w: majority` 의 트랜잭션에서 replica set 으로의 복제가 실패하면 즉시 롤백되지 않을 수 있다. 결과적으로 롤백되거나, 트랜잭션이 적용되거나 둘 중 하나다.
- 몽고 db 5.0 부터는 `coordinateCommitReturnImmediatelyAfterPersistingDecision` 이 설정을 할 수 있다.
    - 기본적으로 적용된다.
    - 적용하면 `shard transaction` 의 경우 커밋이 완벽히 적용되면 클라이언트에게 전달된다.
    - 적용하지 않으면 클라이언트에 반환된 후 롤백될 수 있다.
    - 4.4 이전에는 무조건 기다려야함.

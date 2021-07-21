## Database 

***

### Q. RDBMS vs NoSQL 차이에 대해서

더 나아가면 NoSQL 에서도 Key-Value DB 와 Document DB 이렇게도 있곘지. 

SQL 에는 MySQL PostgreSQL 이런게 있다. 데이터 정규화를 가장 많이 신경쓴다. NoSQL 보다 데이터 구조가 엄격하다. 하지만 NoSQL 의 Document DB 는 이런걸 신경쓰진 않는다. 각 레코드 별로 구조가 다를 수 있다. 
 
대신에 NoSQL 은 빠르고 데이터를 어떻게 뽑을건지에 대한 고민을 한다. 

Scalability 측면에서는 NoSQL 이 더 낫다. 만약 데이터구조가 ManyToOne 이나 ManyToMany 같은 조인을 많이 타야한다면 SQL 이 더 나을 수 있다. 

NoSQL 종류에는 Key-Value DB, Document DB, Graph DB 이런게 있다. 

Document DB 에는 Mongo DB 같은게 있고 Json 기반의 Document 를 저장한다. 

Key-Value DB 에는 Cassandra DB 와 Dynamo DB 가 대표적으로 있다. 카산드라와 Dynamo 는 읽고 쓰기가 엄청 빠르다는 특징이 있다.

Document DB 와 Key-Value DB 를 비교해보면 Document DB 는 여러가지 데이터가 문서화나 그룹화 되어 있다면 Key-Value DB 는 Dictionary 나 Map 과 비슷하다. 그러니까 되게 조회나 write 가 빠르다는 특징이 있다. 

Graph DB 각 요소사이의 관계를 알아야 할 때 Many-To-Many 의 관계가 많을 때 유용하다.  


### Q. Sharding 이란

### Q. Replication 이란 

### Q. Index 에 대해서 

### Q. Transaction 에 대해서 

Transaction ACID

- Atomicity
  - 다 실패하거나 다 중단되는 것. 중간까지만 성공하는 일이 없도록 하는 것
- Consistency 
  - 데이터가 한번 들어가면 변하지 않고 언제나 일관된 값을 유지하는 걸 말한다
- Isolation
  - 트랜잭션이 실행중에 다른 트랜잭션이 끼어들 수 있는 수준을 말한다. 
- Durability
  - 수행된 트랜잭션은 계속해서 반영되어야 함을 말한다. 

@Transactional - propagation

- 실제로 이런 트랜잭션이 AOP로 감싼것 
- @Transactional 설정으로는 ReadOnly, timeout, propagation, isolation, rollback 과 같은 기능을 걸 수 있다. 
- Propagation 설정을 통해서 Transaction 이 적용될 경계를 결정지을 수 있다. 
  - Propagation.REQUIRED 가 기본 설정이다. 활성화된 트랜잭션이 없다면 새로운 트랜잭션을 만들고 새로운 트랜잭션이 있다면 현재의 로직 흐름을 시킨다. 
  - Propagation.SUPPORTED 는 현재 활성화된 트랜잭션이 있다면 그걸 쓰고. 없다면 트랜잭션을 안쓰도록 하는 것. 
  - Propagation.MANDATORY 는 현재 활성화된 트랜잭션이 있다면 그걸 쓰고 없다면 예외를 발생시키는 것 
  - Propagation.NEVER 는 현재 활성화된 트랜잭션이 있다면 예외를 발생시키고 없다면 넘어가는 것
  - Propagation.NOT_SUPPORTED 는 현재 활성화된 트랜잭션이 있다면 잠시 멈추고 비즈니스 로직을 트랜잭션 없이 쓰도록 하는 것 
  - Propagation.REQUIRES_NEW 는 현재 활성화된 트랜잭션이 있다면 잠시 멈추고 새로운 트랜잭션을 만들어서 실행시키는 것. 
  - Propagation.NESTED 는 활성화된 트랜잭션이 있다면 현재 이 지점을 save point 로 잡도록 설정한다. 그래서 비즈니스 로직 중에 예외가 발생하면 이 지점으로 롤백되도록 한다. 
- Isolation 
  - Isolation 은 트랜잭션의 ACID 한 특성중에 하나다. 트랜잭션의 Concurrent 환경에서 어떻게 반영되는지를 표시하는기준이다. 
  - 각각의 Isolation Level 에서 일어날 수 있는 Side Effect 
    - Dirty Read
      - 커밋되지 않은 데이터를 읽는 경우
      - Read Committed 에서 막을 수 있다. 커밋된 데이터만 읽는 걸 허용해서 
    - Nonrepeatable read
      - 한 트랜잭션에서 여러번 읽었을때 값이 Updare 되서 다른 경우 (Inconsistent Read) 
      - Reapeatable Read 에서 막을 수 있다. 선행 트랜잭션이 읽은 데이터를 후행 트랜잭션이 갱신하거나 삭제하지 못하게 하는 것 같은 데이터를 읽었을때 일관성 있는 데이터를 보존해주는 경우 
    - Phantom Read
      - 한 트랜잭션에서 값을 여러번 읽을때 추가된 칼럼이 있거나 지워진 칼럼이 있는 경우
      - Serializable Read 에서 막을 수 있다. 선형 트랜잭션이 읽은 데이터를 후행 트랜잭션이 갱신하거나 삭제하지 못하게 중간에 새로운 레코드를 삽입하는 것도 불가능하게
      
      
***

### Q. B-Tree 란? 

하나의 노드에 여러가지 자료가 배치되는 트리구조.

M 개의 자료가 배치가 가능하다.M이 짝수냐 홀수냐에 따라서 알고리즘이 다르다.

노드의 자료수가 N 이라면 자식의 수가 N + 1 이어야 한다.

각 노드의 자료는 정렬된 상태여야 한다. 그리고 노드의 자료값인 Dk 의 죈쪽 서브트리는 Dk 보다 작은 값이어야 하고 Dk 의 오른쪽 서브트리는 Dk 보다 큰 값들이어야 한다. 

Root 노드는 적어도 2개 이상의 자식을 가져야 한다. 

Root 노드를 제외한 모든 노드는 적어도 M/2 개의 자료를 가지고 있어야 한다.  

MySQL의 DB Engine의 InnoDB 는 B+Tree 로 이뤄져 있는데 이는 B-Tree 의 확장된 개념이다. 

B-Tree 는 균형 트리. 

B+Tree 는 B-Tree 의 확장 개념으로 B-Tree 의 경우 브랜치 노드나 내부 노드에 Key 와 데이터를 담아둘 수 있다. 하지만 B+Tree 의 경우 리프 노드에서만 Key 와 데이터를 담아둘 수 있고 브랜친 노드에서는 Key 만 담아둘 수 있다. 데이터는 넣지 못한다. 

그리고 리프 노드끼리니는 Linked List 로 연결되어 있다. 이를 통해 메모리를 더 확보해서 더 많은 key 들을 수용할 수 있어서 트리의 높이는 더 낮아진다. 

### Q. Index 란? 

데이터베이스에서 검색을 위한 Key 

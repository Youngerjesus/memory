# Spring Cloud Stream property 

https://docs.spring.io/spring-cloud-stream/docs/Brooklyn.RELEASE/reference/htmlsingle/#partitioning

***

## Spring Cloud Stream 의 여러가지 옵션들

### Binding Property

- `spring.cloud.stream.bindings` 에 사용된다.
- ~~Channel 은 source <-> output 이 연결된 걸 말한다.~~
  - 이게 아니라 source -> app, app -> output 을 말함.

### Properties for Use of Spring Cloud Stream

- destination
  - Middleware 의 타겟 도착지.
    - Channel 이 consumer 라면 multiple destination 을 가지는게 가능함.

- Group
  - Channel 의 consumer group
  - Default: null

- Content Type
  - Content 의 Type. 디폴트가 null 이므로 강제하지 않는다.
  - Default: null

- Binder
  - 이 binding 에 사용될 binder
  - Default: null
  - Multiple binder 가 존재할 때 어떤 걸 사용할 지 정해야한다.
  - Binder 가 뭔데?
  - RabbitMQ 와 Kafka 용 binder 가 있는건가? ㅇㅇ 그거임.
  - binder 설정은 global 하게 되거나 각각의 채널마다 binding 을 설정하는 것도 가능하다.
    - `spring.cloud.stream.bindings.input.binder=kafka`
    - `spring.cloud.stream.bindings.output.binder=rabbit`

    
## Consumer Properties

- Concurrency
  - Inbound consumer 의 동시성 수. 리액티브는 무조건 1임.
  - 처리량을 늘릴 때 사용함.
  - Default: 1

- partitioned 가 무엇을 의미하는 걸까?
  - Whether the consumer receives data from a partitioned producer.
  - 데이터를 partitioning 하는 건 가능함.
  - 하나 또는 그 이상의 multiple producer 가 multiple consumer 에게 보내는 것도 가능.
  - Consumer 는 common characteristics 로 데이터를 식별하고, 이를 통해 같은 컨슈머가 처리한다.
  - True 라면 consumer 는 여러 인스턴스들에 파티션 된다.
  - 이렇게 쓰인다.
    - `spring.cloud.stream.bindings.input.consumer.partitioned=true`
    - `spring.cloud.stream.instanceIndex=3`
    - `spring.cloud.stream.instanceCount=5`
    - instanceCount 는 데이터가 partition 되는 상황에서 어플리케이션의 전체 수를 말한다.
    - instanceIndex 는 여러 인스턴스들 중에서 유니크해야 하는 값이다. 0 ~ instanceCount - 1 값이어야한다.
      - 이 걸 바탕으로 파티션을 구별할 수 있게 해준다.
  - Spring Cloud Data Flow 는 이를 런타임 인프라 시점에 가능하게 해준다.
  - 생각해보면 파티션은 자동적으로 할당되었던 듯.
  - Default: false

- instanceIndex
  - 매칭될 파티션 넘버. 

- instanceCount

- Header Mode
  - Raw 라면 헤더 파싱은 하지 않는다.
  - non-Spring Cloud Stream applications 에서 유용한다는듯.
  - Header 는 어디에 쓰이는데?
    - Producer 가 생산한 정보의 컨텐츠 타입을 저장할 때 contentType 을 헤더에 붙인다.
    - 메시지를 다룰려면 contentType 이 있어야함.
  - Default: embeddedHeaders

- backOffInitialInerval
- backOffMaxInterval
- backOffMultiplier


## Producer properties

- partitionKeyExpression
  - 설정하면 outbound data 가 이 채널에 partition 됨. 
  - partitoin Enable 의 유무라고 생각하면 됨. 
  - 이때 partitionCount 는 1보다 커야한다. 효과적으로 쓰기 위해서.
  - 이런식으로 쓴다.
    - `spring.cloud.stream.bindings.output.producer.partitionKeyExpression=payload.id`
    - `spring.cloud.stream.bindings.output.producer.partitionCount=5`
  - partitionKeyExpression 가 파티션을 결정할 떄 쓰인다.
  - 결정되는 번호는 0 ~ partitionCount - 1 임.
  - 기본으로 쓴다면 `key.hashCode() % partitionCount` 가 됨.

- partitionKeyExtractorClass
- partitionSelectorClass
- partitionCount
  - Data 를 파티션하는 수.
  - Partition 가능하게 한 후 이것도 해야 파티션이 잘 됨.
  - Target Topic 의 파티션 수와 이 값이 사용된다.

- 파티션은 카프카의 파티션과 똑같이 생각하면 되나? ㅇㅇ
  - 파티션 처리를 지원한다. 그리고 이 부분을 추상화해놨다.
  - 카프카처럼 되는 건 맞는듯. Rabbitmq 에선 지원하지 않고.

- Actuator 를 넣으면 바인딩 정보도 볼 수 있는듯.


## Kafka Binder Properties

- `spring.cloud.stream.kafka.binder.brokers`
  - Kafka 가 연결될 Broker 의 리스트.

- `spring.cloud.stream.kafka.binder.offsetUpdateTimeWindow`
  - Offset 이 save 될 주기. 0 으로 셋팅하면 무시된다.
  - Default: 10000

- `spring.cloud.stream.kafka.binder.offsetUpdateCount`
  - 컨슘된 Offset 이 update 되는 빈도의 수. 0 으로 셋팅하면 무시된다.
  - offsetUpdateTimeWindow 과는 상호 배타적이다.
  - Default: 0 (업데이트를 여러번 하는 거 아닐까?)


## Kafka Consumer Properties

- autoRebalanceEnabled
  - Consumer member 들 그룹사이에 자동으로 리밸런싱된다.
  - 이걸 안쓰면 고정된 partition 에 할당해야함. 
  - 여기에 `spring.cloud.stream.instanceCount` 이것과 `spring.cloud.stream.instanceIndex` 이게 쓰인다. 
    - 이걸 기반으로 파티션에 연결됨. 그리고 쓸거면 instanceCount 는 1보다 늘 커야되고, 실제 파티션도 1보다 많아야함.
  - Default: true

- autoCommitOffset
  - 메시지가 처리될 때 auto commit 을 할 것인지.
  - Default: true

- autoCommitOnError
  - autoCommitOffset 이 true 여야만 적용된다.  
  - 이 값이 False 인 경우 성공한 메시지만 commit, 실패한 메시지는 커밋하지 않는다. 그래서 리플레이가 가능하다.
  - True 인 경우에는 error 가 나도 커밋을 한다.
  - Not set 인 경우 (이게 default) enableDlq 에 따라서 다르다. DLQ 에 보내진 경우에만 auto commit 을 한다.
    - DLQ (Dead-letter queue) 는 처리가 실패한 메시지들만 가지고 있는 queue.
    - 이 큐의 목적은 처리가 실패된 메시지들을 가지고 있어서 분석할 수 있고 필요하다면 액션을 하기 위해서.
    - Secondary topic 이다.
    - 컨슈머가 처리를 실패할 때 자동으로 이 큐에 보낸다.  
  - Default: not set.

- enableDlq
  - True 로 설정한 경우 DLQ behavior 로 보낸다.
  - 에러가 나도 Replay 를 하지 않는 방법인듯.
    - 에러가 나도 괜찮거나, replay 처리가 비용이 큰 경우에는 이게 낭르 수 있으니.
  - 아 기록은 해놓고 replay 는 하지 않는거구나.
  - False 는 이걸 안쓰는 거고.
  - Default: false

## Kafka Producer Properties

- bufferSize
  - 카프카 프로듀서가 보내기전에 배치할 수 있는 데이터 양.
  - Default: 16384 byte.

- Sync
  - Producer 가 동기식인지.
  - Default: false

- batchTimeout
  - 메시지를 보내기전에 축적해서 보내는 것.
  - 0 보다 크다면 latency 를 희생해서 처리량을 올린다.
  - Default: 0 

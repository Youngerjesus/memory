## RabbitMQ vs Apache Kafka

메시지 브로커 서비스로 RabbitMQ 와 Apache Kafka 둘 다 사용이 가능했다. 

Apache Kafka 정의 자체가 Distributed Event Streaming Platform 로 Routing 처리보다는 
연속적인 데이터 처리에 특화되어있다. I/O 를 할 때도 Linear 방식을 사용하니까 Random Access 보다
성능이 좋고. 

- Linear 방식을 사용하기 위해서는 append-only 방식을 사용한다./ 

#### 왜 RabbitMq 와 Kafka 사이에서 Kafka 를 골랐는지. 

RabbitMQ 는 AMQP 프로토콜을 이용한 복잡한 Routing 처리에 특화되어있었고 Apache Kafka 는
Disk I/O 를 할 때 Random Access 가 아닌 Linear Access 로 파티션 파일에 접근할 때 성능이 좋았고 
OS 의 PageCache 를 이용하고 linux 의 sendFile 이라는 시스템 콜을 이용해서 pagecache 로부터 바로
socket 으로 데이터를 복사하는 방법을 쓰므로 Byte Copy 를 줄이는게 가능하기  떄문에 성능상에 우위가 있다는
특징이 있었다. 

두가지를 섞어서 쓰는건 이해하고 쓰기엔 충분한 시간이 없었다. 그러므로 한가지만 고른다고 했을때 RabbitMQ 는 
Queue 에서 메시지를 뽑아가면 그 자체로 끝인데 Kafka 는 log 로 며칠동안 남겨지니까 Durability 가 더 높으니까
장애가 날때 대응이 더 쉽지 않을까 라는 생각이 있어서 선택했다. 
 


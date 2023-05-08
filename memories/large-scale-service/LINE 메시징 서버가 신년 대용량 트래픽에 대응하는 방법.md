## LINE 메시징 서버가 대용량 트래픽에 대응하는 전략

이 글은 라인의 Tech Youtube 와 Tech Blog 를 기반으로 작성했습니다.

<1. 기본적인 Connection 관리 전략>

매번 요청할 때마다 Connection 을 맺지 않고, HTTP 와 TCP Keep-Alive 를 사용하여 Connection 을 유지해 성능 향상을 이룹니다.

Proxy 와 어플리케이션의 WAS 가 연결된 구조에서는 Proxy 단에서 WAS 에 전달할 수 있는 MaxConnection 을 제한해 놓습니다. 이를 통해 WAS 에서 Connection 이 부족해 처리를 못하는 상황을 방지하고, Connection 이 급증하여 성능이 저하되는 문제를 해결합니다.
- 예전에 LINE에서는 Connection 수가 급격히 증가한 경우 해당 컴포넌트의 힙(Heap) 사용률이 빠르게 상승하여 Full GC 가 발생했고, 이로 인해 성능 저하가 크게 나타났다고 합니다.

<2. 트래픽 패턴 분석 및 서버 증설, 벤치마크 테스트>

라인은 Redis 를 사용하며, 과거 데이터를 기반으로 트래픽을 분석하고 예측합니다.

트래픽 패턴을 분석할 때 Redis 의 IOPS (Input/Output Operations Per Second) 와 UsedMemory 수치를 참고합니다.
- IOPS 는 초당 요청 횟수를 의미하며, 신년과 같이 사용자의 요청이 급증할 때 증가하는 수치입니다. Redis 클러스터가 GET 요청을 처리하기에 충분한지 확인하기 위해 참고합니다.
- UsedMemory 는 Redis 에 저장된 데이터의 크기를 나타냅니다. PUT 요청이 많을 경우, UsedMemory 수치를 주의 깊게 살펴야 합니다. 요청량이 급증하여 사용 가능한 최대 메모리를 초과하면 서비스에 큰 영향을 미칠 수 있습니다.
- 과거 데이터를 바탕으로 필요한 메모리 크기를 산정한 후, 증설이 필요한 경우 증설 작업을 진행합니다.

벤치마크 테스트는 트래픽을 서서히 증가시켜보는 것과 함께, 한 번에 많은 트래픽을 부여하는 경험도 필요하다고 합니다.

<3. Redis 클러스터의 안정적이고 효율적인 사용>

라인은 Redis 를 캐시 뿐 아니라 주요 저장소로 사용하고 있습니다. 주요 저장소는 데이터 수명이 긴 것들을 보관합니다. 영구 저장소로는 HBase를 사용하고 있습니다.
- 60 개 이상의 Redis 클러스터를 사용하며, 각 클러스터는 1000-3000 개의 노드를 포함합니다.
- 1000 개 이상의 물리적 머신을 사용하고, 각 머신에는 10-20 개의 코어와 192-256GB 의 메모리가 포함되어 있습니다.

빠른 응답 속도를 위해 Client Side Sharded 된 구조로 Redis 클러스터를 사용합니다.
- 프록시를 사용하지 않아 더 빠른 응답 속도를 얻을 수 있으며, 응답 속도는 100-200 마이크로초 정도입니다.
- 샤딩 규칙으로는 Fixed size ring 또는 Consistent Hashing 을 사용합니다.

Dual Write 를 통해 쓰기 작업의 지연 시간을 줄입니다.
- Redis 에 쓰기 작업을 수행한 후 응답을 받으면 HBase 에 기록합니다. HBase 의 기록은 백그라운드에서 수행되며, 실패할 경우 Kafka 에 기록합니다. 이후 Kafka 를 통해 비동기적으로 처리됩니다.

읽기 작업에 고가용성(High Availability)을 부여합니다.
- 먼저 Redis 에 읽기 요청을 보내고, 몇 백 마이크로초를 기다려도 응답이 오지 않으면 HBase 에 읽기 요청을 보냅니다. 먼저 도착하는 쪽을 응답으로 사용합니다.
- 아주 가끔 Redis 가 다운될 수 있기 때문에 이러한 방식을 사용합니다.

Hot Key 문제를 해결하여 안정적인 운영을 달성합니다.
- Hot Key 문제란 일부 키에 대한 액세스 빈도가 다른 키에 비해 압도적으로 높은 경우를 말합니다. Hot Key 가 있는 경우 특정 서버나 노드의 성능이 급격히 저하됩니다.
- Write Intensive 한 Hot Key 의 경우 키를 재설계(redesign)합니다.
- Read Intensive 한 Hot Key 의 경우 동일한 키를 여러 클러스터에 복제하여 서비스합니다. 이를 Repliacted cluster 라고 하며, 애플리케이션은 읽기 작업을 수행할 때 이러한 클러스터 중 하나를 무작위로 선택하여 응답을 받습니다.

Timeout 과 Circuit Breaker 설정을 통해 시스템의 안정성과 성능을 유지합니다.
- 타임아웃을 짧게 설정하여 분산 시스템에서의 안정성을 유지합니다.
- 타임아웃만으로는 부족하여 서킷 브레이커를 도입합니다. 서킷 브레이커를 통해 Fast Failure 처리 할 수 있으며, 이를 통해 바쁜 Redis 서버에 요청을 보내지 않게 합니다.

다양한 역할을 담당하는 Redis 클러스터를 분리하여 별도의 클러스터로 구성합니다.
- LINE의 Redis 클러스터는 지속적으로 트래픽이 증가하므로, 안정적으로 트래픽을 처리하기 위해 성격이 다른 데이터를 별도의 클러스터로 구축하여 이동시킵니다.

비정상적인 호출을 하는 악성 사용자를 파악하여 차단하는 기능을 추가합니다.
- 행동 패턴을 분석해 악성 사용자를 차단하고, 클러스터의 IOPS 를 안정화시킵니다.

Redis 클러스터에서 TTL(Time To Live) 을 추가하고 Eviction Policy 을 설정합니다.
- UsedMemory가 지속적으로 증가하는 클러스터의 경우, TTL(Time To Live)을 지정하여 큰 문제가 없는 데이터에 대해 TTL을 설정하고 제거 정책을 'volatile-ttl'로 변경합니다.
- 'volatile-ttl' 정책은 expire 필드가 true로 설정된 키의 경우, TTL이 가장 짧게 설정된 키를 삭제하는 정책입니다.

References:
- https://engineering.linecorp.com/ko/blog/how-line-messaging-servers-prepare-for-new-year-traffic
- https://engineering.linecorp.com/ko/blog/presenting-on-redisconf18
- https://www.youtube.com/watch?v=IxTUXuoIWro
- https://www.youtube.com/watch?v=5oTlFJ0llNw&t=308s
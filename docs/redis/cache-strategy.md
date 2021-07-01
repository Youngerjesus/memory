## Cache Strategy 

***

#### Cache-aside (Lazy-loading)

- 먼저 캐시에서 데이터를 찾고 없으면 DB 에서 가지고 오는 것 

#### Write-Behind

데이터를 캐시에 먼저 쓰고 그 다음 비동기식으로 데이터를 업데이트하는 방법.  

#### Writhe-Through

캐시와 데이터베이스 동기식으로 업데이트 하는 방법. 데이터의 일관성을 유지하는데 좋다. 

#### Read-replica

데이터의 고가용성을 위해서 또는 읽기 전용 쿼리 (예를들면 느린 O(n) 쿼리)에 사용하기 위해서 Replication 을 사용한다. 
 

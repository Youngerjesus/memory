## Redis 와 Memcached 비교 

모두 in-memory 기반이라는 것 Key-Value 형식의 No-SQL 이라는 것

하지만 다양한 데이터 스트럭처를 레디스는 지원하고 한개의 키당 최대 512MB 의 값을 저장한다 라는 것 

하지만 Memcached 는 최대 1MB 의 크기제한이 있다라는 것 그래서 복잡한 구조를 넣어야 하는 경우에는 Redis 가 더 낫다
 
Memcached 는 멀티 스레드 기반으로 다중 코어를 이용할 수 있다라는 것 
 
Redis 는 싱글 스레드를 기반으로 처리한다는 것. 

컴퓨팅 리소스를 이용해서 더 나은 성능을 발휘하는건 Memcached 가 더 낮고  

Redis 는 싱글 스레드 기반으로 사용하니까 O(n) 명령어 를 사용할 때 주의해야 한다라는 것 


#### flush 가 다르다. 

Memcached 는 lazy delete 를 하지만 Redis 는 즉시 모두 지운다.
# Redis 잘쓰고싶다.

## Redis 는 싱글 스레드

Redis 는 싱글 스레드이므로 다음과 같은 명령은 주의하자.

> 1) 전체 키 명령을 조회하는 keys 명령은 사용하지 말자.

2) Redis 메모리를 비우는 flushall 과 flushdb 명령은 주의하자.
>

이 명령들은 대체할 수 있다.

- `keys` → `scan` 으로 대체 가능
    - `scan` 은 일정한 개수만큼 키를 조금씩 가지고와서 처리하는 방식이다.
    - 주로 패턴 매칭과 파이프라이닝을 이용해서 많이 사용한다. 예) user* 라는 키를 가진 데이터를 모두 지워줘
- `flushall` 은 `flushall async` 로 대체 가능 (redis 4.0.0 부터 가능)
    - `flushall async` 는 background 방식으로 처리한다.

이외에 적재를 할 때 사용하는 [`lazyfree`](http://redisgate.kr/redis/configuration/param_lazyfree.php) 옵션을 주는 것도 Redis 입장에서 좋아보인다.

- Redis 의 메모리가 가득찰 때 maxmemory 를 설정한 경우 maxmemory-policy 를 따라서 키를 지운다.
- `lazyfree` 는 이것들은 동기식으로 처리하는게 아니라 백그라운드에서 지우는 옵션이다.

## Redis 는 fork 를 하게될 때 최대 메모리를 두 배까지 쓸 수 있다.

~~Redis 는 인메모리 데이터 스토어인데 주어진 메모리보다 더 많이써서 스왑이 발생하면 안된다.~~

최대 메모리를 두 배까지 사용하는 이유는 fork 를 할 때 사용하는 기술인 COW (copy-on-write) 이다.

Fork 를 할 경우 부모와 자식 프로세스는 읽기 메모리는 공유하고 **쓰기 메모리만 복사**해서 사용하는 것.

그렇다면 언제 fork 를 하게될까?

- 데이터 갱신 작업을 할 때 해당 데이터만 잠시 두 배가 된다.
- master 에 replica 가 연결되는 순간 데이터 이전을 위해 RDB 를 만들면서 한다.
- AOF Rewrite 를 위해 `bgrewriteoaof` 명령을 내리면 한다.
- RDB 생성을 하기 위한 경우 (bgsave 인 경우) 에 한다.

이 요소들을 고려해서 redis 의 maxmemory 설정을 하면 될 것 같다.

- rdb 를 사용하는 경우에는 maxmemory 를 45% 로 사용하는 걸 권장하고 이것들을 쓰지 않으면 최대 80% 로 설정하라고 한다. (물론 다른 프로세스가 사용하는 메모리도 생각해서 설정하면 될 것 같다.)

## 유익한 Redis 의 Monitoring Metric

redis 에 접속해서 `info all`  명령을 입력하면 볼 수 있다.

| 대항목 | 항목 | Description |
| --- | --- | --- |
| memory | used_memory_rss | Redis 가 현재 사용하고 있는 물리 메모리 양. 실제 메모리 사용량이 많으면 swap 이 일어나서 성능이 떨어진다.  |
|  | used_memory | redis 가 요청한 메모리의 양 |
|  | mem_fragmentation_ratio | used_memory / used_memory_rss 비율로 이 값이 높다면 fragmentation 이 높다고 보면 되고 1 보다 적다면 swap 이 발생하고 있다는 것. |
| stats | instantaneous_ops_per_sec | 초당 실행하는 명령 수. Redis 는 CPU 의 영향을 많이 받으므로 더 좋은 CPU 를 쓰면 더 많은 처리를 한다. |
|  | total_commands_processed | 지금까지 처리한 명령 수 |
|  | expired_keys | 지금까지 expireation 이 발생한 이벤트 수  |
|  | keyspace_hits | 캐시 Hit 한 수 |
|  | keyspace_misses  | 캐시 miss 한 수  |
| clients | connected_clients | 협재 접속해 있는 클라이언트 수. Redis 는 싱글 스레드라 클라이언트가 지속적으로 연결/해제 하면 성능이 떨어질 수 있다. 이 값이 크면 성능에 악영향을 주고 있다는 뜻.  |
|  | maxClients | 접속할 수 있는 최대 클라이언트 수  |
| replication | master_repl_offset | Primary 의 replication offset  |
|  | slave_repl_offset | Secondary 의 replication offset 으로 replication 에서만 존재한다.
(replicatoin lag = master_repl_offset - slave_repl_offset 를 의미한다.  |
| Commandstats | cmdstat_XXX | Redis 가 해당 명령을 처리하는 속도를 말하며 1 에 가까워야 한다. (1 은 1 us 를 말한다)  |

이외에도 알아야 하는 지표들이 있다. ``Slowlog`` 와 ``latency`` 이것들은 redis.conf 에 파라미터로 지정하면 파일로 로그를 남겨준다고 한다. 방법은 여기를 [참고](http://redisgate.kr/redis/server/server_monitor.php)
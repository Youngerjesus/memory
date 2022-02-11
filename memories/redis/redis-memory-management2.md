# Reds for management 2

## Swap 이 발생했을 경우에 할 수 있는 조치.

- 스케일 업.
- engine parameter 로 Redis cluster 에 Reserved Memory 를 설정하는 것.
- 재시작 하는 것도 방법이긴 할 듯.

## Reserved Memory 설정이 뭔데

- Redis cluster 를 사용하는 경우 + write heavy 인 경우 50% 로 설정해두는 걸 권장한다.
- 기본적으로 reserved-memory 값은 0 이다. 그리고 reserved-memory-percent 라는 설정도 있다.
- AWS elasticcache 의 경우 reserved memory-percent 설정을 25% 로 해놓는다.
- backup 과 failover 를 위해서 설정하라는 말이 뭘까? 파일로 적는거 아님?
    - rdb 를 할 때 자식 프로세스를 만들어서 자신의 현재 메모리 내용을 파일로 기록한다. 그래서 추가적인 메모리 사용이 발생할 수 있다.
    - rdb 가 실패하면 그 이후의 write 명령은 모두 무시된다.
    - Persistence 성은 없고 캐시 용도로만 쓴다면 RDB 를 사용하지 않는 것도 방법이다.
- Reserved memory 설정은 AWS Elastic cache 에만 있는거 같은데 Redis 에서는 maxmemory 설정만 있는듯.

## Maxmemory

- maxmemory 설정만큼 redis 를 쓰는듯.
- maxmemory 이상을 쓰려고 할 때 OutofMemory Error 를 내도록 하는듯. 이걸 안쓰면 스왑까지 사용해서 작업을 할거니까.
    - 더 자세히 말하면 eviction 을 할 수 있는지 보고 할 수 있다면 하고나서 작업을 하고 아니면 OOM 을 내는 거 같음. 맞네 EVICT_FAIL 이 나면 이제 OOM 에러를 낸다.
- maxmempry-policy 도 보는게 좋을듯. 기본값이 noeviction 인거 같다. (제거를 안하는듯)
    - noeviction
        - 캐시를 지우지 않는 정책
    - allkey-lru
        - LRU 알고리즘 기반으로 키를 삭제한다.
    - allkey-random
        - 랜덤하게 키를 삭제한다.
    - allkey-lfu
        - LFU 알고리즘 기반으로 키를 삭제한다.
    - volatile-lru,lfu,random
        - EXPIRE SET 에 있는 키를 LRU,LFU,RANDOM 기반으로 키를 삭제한다.
- maxmemory 에 도달하면 Out of memory 에러를 일으켜서 응용프로그램에 오류가 발생하지만 전체 시스템이 중지되지는 않는다.
- maxmemory 는 32 bit 기준으로 3GB 로 등록되고 64 bit 기준으로는 0 으로 등록된다. 즉 스왑까지 사용한다.
- maxmemory 를 사용하면 swap 을 사용하는가?
    - ㄴㄴ
- maxmemory 를 사용하면 남는 메모리를 write 작업이나 persistence 작업에 사용하는가?
    - ㄴㄴ

## LazyFree

- 빅데이터 쓰기 작업을 하는 경우 Max 메모리에 도달하게 되는 경우 이떄 키를 연속적으로 삭제하면 지연 문제가 생겨서 막아주는 설정인듯?
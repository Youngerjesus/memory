# How does the WiredTiger cache eviction process work

https://muralidba.blogspot.com/2018/03/how-does-wiredtiger-cache-eviction.html

## Default WiredTiger cache size

- WiredTiger 내부 캐시
    - RAM * 50% - 1GB or 256MB

## The eviction process

- 일단 캐시에 있는 데이터를 2종류로 구분
    - clean
        - disk 에 있는 데이터와 메모리에 있는 데이터가 서로 동일.
    - dirty
        - disk 에 있는 데이터와 메모리에 있는 데이터가 서로 다름. 그래서 주기적으로 (60초마다) 플러쉬를 함.

## Eviction triggers

1) The cache is at or beyond a specific threshold.

- WiredTiger 는 캐시의 사용량을 80% 로 미만으로 두려고 한다. (기본적으로.) 그래서 이것보다 넘어가면 eviction 이 실행됨
    - eviction trigger 되는 비율을 80% 보다 높게 설정하는 것은 권장하지 않음. 어플리케이션 스레드가 eviction 과정에서 동참해서 느려질 수 있음.
    - 그리고 cache 에서 dirty 비율이 높아진다면 (20%) 클라이언트의 요청을 처리하기 전에 evict 과정부터 먼저 수행함.
    - 이떄는 worker thread 에 의해서 이뤄진다.
- cache 사용량이 95% 에 도달하면 어플리케이션 스레드도 eviction 과정에 동참한다.

2) The cache contains too much dirty data.

- dirty 비율이 캐시의 5% 정도에 도달하면 cache eviction process 가 시작한다.
- 이건 checkpoint 과정을 할 필요가 없도록 하며 checkpoint 과정 자체가 느려지도록 만들어서 정지한 것처럼 보이게 만든다.

3) The page grows too large.

- bulk insert 혹은 bulk update 로 인해서 in-memory page 가 threshold (default: 8MB) 에 도달한다면 in-memory page 는 쪼개지고 해당 페이지는 eviction 된다.
    - (과도한 양의 b-tree 가 만들어져서 그런가)

## Eviction processes

- Worker threads
    - wiredTiger engine 에 있는 eviction process 에 의해서 수행함. 기본 스레드는 4개
- Application threads
    - MongoDB 와 연결된 client thread 로 부터 시작함.
    - 만약 eviction 이 필요한 과정이라면 (캐시가 95% 이상 쓰이고 있어서.) 어플리케이션 Operation 이 수행되기 전 eviction 부터 먼저 수행된다.

## Etc

- checkpoint 과정과 dirty page 가 5% 넘어서 disk 로 옮기는 과정은 동일하다.
    - 참고: https://www.mongodb.com/community/forums/t/internals-checkpoints-and-journaling/6270
- WiredTiger Storage Engine 에서 In-memory Page Size 를 튜닝할 수 있다.
    - 일정 크기를 넘어가면 페이지가 쪼개지는 둣. (Default: 5MB)
    - 페이지가 쪼개지면 각 페이지에 접근할 때 exclusive lock 을 얻어야 함. 이거 (쪼개지는 시간, exclusive lock 을 이용하는 시간) 떄문에 어플리케이션이 기다릴 수 있음.
    - 크기를 작게 만들면 자주 쪼개지고 대신에 lock 을 쓰는 시간이 짧음.
    - 크기를 키우면 lock 을 이용하는 시간이 길 수 있음. 대신에 페이지가 자주 쪼개지지 않음
    - 참고: http://source.wiredtiger.com/mongodb-3.4/tune_page_size_and_comp.html

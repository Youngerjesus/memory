# Performance Best Practices: Hardware and OS Configuration

https://www.mongodb.com/blog/post/performance-best-practices-hardware-and-os-configuration

***

## Run on Supported Platform

- 몽고 DB 를 다양한 환경에서 돌릴 수 있는데 이 링크를 참조해서 어떤 환경에서 어떤 버전까지 이용할 수 있는지 확인해야한다.

    - https://www.mongodb.com/docs/manual/installation/?_ga=2.174892518.550717159.1663043268-583274204.1651135057#supported-platforms

## Ensure Your Working Set Fits in RAM

- **MongoDB 는 working Set (index 와 자주 사용하는 데이터) 가 메모리에 모두 있을 때 최고이다. 그래서 메모리 사이즈가 중요함.**

- RAM 이 충분하지 않으면 절대 안된다.

## Use Multiple CPU Cores

- WiredTiger Storage Engine 은 Multiple CPU Cores 를 사용하는데 적합하다.

- 사용자의 커넥션은 하나의 스레드로 배치된다. 그리고 백그라운드 스레드로 Cache eviction 을 하거나 checkpointing 을 한다.
    - checkpointing 은 recovery 를 위해서 snapshot 들을 저장하는 공간이다.
        - 이 스냅샷을 기준으로 일관된 데이터를 읽을 수 있다.
        - MongoDB 는 MVCC (MultiVersion Concurrency Control) 구조를 가진다.
        - 몽고 DB 는 60초 주기로 checkpoint 를 한다. (3.6 이후부터.)
        - checkpoint 의 유효지점은 완전히 새 것이 만들어졌을 때다. 새 checkpoint 를 만들고 있을 때, 에러가 나서 다시 재시작할 때 는 이전 checkpoint 가 유효하다.
        - 새 checkpoint 가 쓰이는 시점은 WiredTiger metadata table 에서 아토믹하게 업데이트가 되었을 때이다.
        - last checkpoint 이후부터 지금까지 변경 사항까지 모두 적용하려면 journaling 을 이용해야한다.
    - **Wired Storage Engine 은 데이터를 캐시에 저장해두고 최대 사이즈에 도달하면 eviction 하는 정책이 있다.**
        - 이걸 어떻게 정하느냐에 따라서 latency 가 줄어들 수 있고, 어플리케이션 처리량에도 도움을 줄 수 있다.
        - 이 정책을 바꿀려면 첫 시작할 때 바꿔야하는듯.
        - `eviction_target` 설정은 캐시를 유지할려는 비율을 말하고 기본 값은 80% 이다.
        - `eviction_trigger` 는 캐시를 제거하기 위한 application thread 가 동작하는 기준을 말하며 기본 값은 95%이다. 이 작업 중일 때 throttle 당하며 latency 가 늘어난다.
        - cache 사이즈가 100% 에 도달하면 application 의 operation 은 모두 멈춘다. 이 경우에는 미리 eviction thread 를 늘려서 막도록 하거나, 메모리를 늘리고 더 많은 캐시를 주도록 해서 막는 방법이 있다.
        - `eviction_dirty_target` 의 기본 값은 5% 이고 `eviction_dirty_trigger` 의 기본 값은 20% 이다. 더티 페이지가 20%에 도달하면 어플리케이션 스레드는 throttle 당한다.

- 그 다음에 더 많은 RAM 과 disk IOPS 를 고려하는건 데이터베이스 성능에 좋다.
    - IOPS (Input/Output Operation Per Second) 를 말한다.
        - 이는 HDD, SDD, NVMe 등 저장장치의 속도를 나타내는 단위다.
        - IOPS 는 Random Access 와 Sequential Access 에 따라 다르다.
        - 계산은 `초당 데이터 전송량 = IOPS * 블럭크기` 로 계산한다.
        - AWS EBS 볼륨 기준으로 SSD 는 16,000 이고 HDD 는 500 이다.

- Atlas 에서 환경을 구축한다면 이 정보를 보고 maxConnection 개수를 제한하자.
    - https://www.mongodb.com/blog/post/performance-best-practices-hardware-and-os-configuration

## Dedicate Each Server to a Single Role in the System

- 같은 replica set 에 있는 멤버를 같은 physical 장비에 두지마라. (failover 를 위한 당연한 말.)

## Configuring the WiredTiger Cache

- `storage.wiredTiger.engineConfig.cacheSizeGB` 설정을 통해 wiredTiger Storage Engine 에 적용되는 캐시값을 설정할 수 있다.
    - **캐시 기본 값은 RAM 의 50% 와 RAM - 1 GB 중 더 큰 양을 선택한다.**
    - 이 값을 조정할 땐 주의하자. 오히려 성능이 떨어지는 케이스가 있을 수 있음.

## Use Multiple Query Routers

- **라우팅 프로세스인 mongos 를 많이 쓰는게 좋다.**
    - 고가용성과 확장성을 위해서.
    - 근데 너무 많으면 mongo config 서버와의 통신 떄문에 config 서버의 성능이 떨어질 수 있음.

## Use Interleave Policy on NUMA Architecture

- 메모리 아키텍처 중 Numa 를 쓰고 있다면 Interleave 를 사용해라.
    - NUMA 를 쓸 때 문제가 있다는 듯.
    - Interleave 는 골고루 메모리 분배를 받는 방식임.

## Network Compression

- 네트워크 트래픽 비용을 낮추기 위해서 Network Compression 을 사용하라.

## Storage and Disk I/O Considerations

- 모든 데이터를 메모리에서만 읽어오는게 아니기 때문에 disk 장비도 중요하다.
- SSD 를 쓰는게 좋다. MongoDB 같은 데이터베이스는 Sequential READ 만 하는건 아니므로.
- **여러가지 장비를 관찰해봤는데 (SATA, PCIe, and NVMe SSDs.) SSD 가 가장 낫다. (read heavy application 에서. working set 을 모두 메모리에 올릴 수 없다면.)**
- RAID-10 구조를 가용한다.

## Use MongoDB’s Default Compression for Storage and I/O-Intensive Workloads

- MongoDB 는 기본적으로 snappy 압축 알고리즘을 써서 50% 정도 스토로지를 절약하고, IOPS 성능을 높인다.
    - documents 와 index 모두 압축이 적용된다.
    - **index 는 또 prefix compression 을 사용해서 메모리에서 사용한다.**
    - snappy 는 50% 정도의 압축과 적은 cpu 를 사용하도록 하는게 특징이다.
    - zstandard 와 zlib 는 더 많은 압축을 떙긴다. cpu 를 더 써서. 

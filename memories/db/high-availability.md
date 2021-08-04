## MySQL 이중화 

Reference: https://www.youtube.com/watch?v=dCVKAJ7tb70
***

### DB 이중화의 필요성 

데이터베이스가 싱글로 사용하는 경우 장애가 발생하는 경우에 장애를 해결하기 까지 서비스로 들어온 데이터를 저장할 수 있고 
서비스는 요청을 처리해줄 수 없다. 그러므로 장애가 발생할 때 생기는 비용인 너무크다. 

#### 개선 1) Master-Slave 구조 

마스터 데이터베이스에서 장애가 나는 경우에 서버의 데이터베이스 커넥션 정보를 슬레이브 데이터베이스로 옮기고 배포 함을 통해서 
장애시간을 단축하고 문제를 해결할 수 있다. 
- 이 경우 재배포 시간까지 장애가 이어질 수 있으므로 아직도 비용이 크다. 

#### 개선 2) Master-Slave 구조 + VIP(Virtual IP)

마스터 데이터베이스 앞단에 VIP 를 두고 서버의 데이터베이스 커넥션에서 VIP 를 통해서 연결한다. 그래서 장애가나면
VIP 를 슬레이브 데이터베이스와 매핑 시킨다면 재배포를 하지 않아도 되니까 좀 더 낫다.  
- 이 경우 마스터 데이터베이스에 문제가 생길때 바로 바꿔줘야 하니까 Health Check 가 필요하다. 이 방법을 통해서
장애가 나면 FailOver 가 일어나게 한다. 

### 이중화 방안

#### HW 이중화 - SHARED DISK 방식 

Master Active - Master Standby 를 두고 Master Active 가 장애가나면 Standby 로 대체하는 식
Standby 는 FailOver 를 위해서 사용만하니까 비용이 크다. 

- 이 방법은 고비용의 SHARED DISK 가 필요하다

#### HW 이중화 - DISK 복제 방식

Master ACTIVE 서버에서 데이터 업데이트가 나면 네트워크를 통해서 Master Slave 에서 동기화 작업을 해준다.
마찬가지로 Master Active 가 장애가나면 Slave 가 대체를 한다. 

- SHARED DISK 방식에 비해 license 나 고성능의 DISK 없이 사용이 가능하다.

- 네트워크 latency 에 의한 성능 영향을 받는다는 단점이 있다.  

- HW 이중화 전체의 문제로는 StandBy 는 FailOver 시에만 사용이 가능하다라는 단점이 있다. 가성비가 생각보다 안나온다. 

#### MySQL Replication 이중화

Binary Log: Master 에서 실행되는 모든 DML/DDL 을 기록해놓는다.

Relay Log: Binary Log 에 있는 내용을 Slave 에 기록한다. 

주로 동작하는 방식은 Slave 에서 추가된 Binary Log 가 있는지 API 를 통해서 확인하고 있다면 이를 가지고 와서 
데이터 동기화를 한다.

MySQL Replication 에서 데이터의 일관성을 위해서 사용자가 마스터 데이터베이스에 업데이트 요청이 가게되면 Replication 에서 Relay Log 를 받고  그거에 대한 
쿼리를 실행을 하고 완료했다는 Ack 를 날려주면 최종적으로 사용자에게 OK 가 가능 구조로 만드는게 가능하다.   
- semi-synchronization 을 이용한 것 

#### MySQL Replication - Multi-Slave 환경 

슬레이브 중 하나만 ACK 가 오면 클라이언트에게 완료를 리턴하므로 슬레이브의 데이터 일관성을 보장하긴 어려울 수 있다.

#### MySQL Replication - MMM(Multi-Master Replication Manager)

Perl 기반의 Auto Failover Open Source

MMM Monitor 에서 DB 서버의 Health Check 와 Failover 를 수행한다. 

MMM 은 Master-Active 와 Master-Standby 가 양방향 복제를 하는 방식으로 되어있다. 

- Master-Standby 에 데이터를 직접적으로 쓰지 못하게 ReadOnly 로 지원한다. 

- Slave 가 있다면 Master 와 Slave 사이에서는 마스터에서 슬레이브로 단방향 데이터 복제를 만들 수 있다. 

Master-Active 가 작동하지 않는다면 MMM Monitor 에 의해 Failover 가 작동한다. 

1. Master Active 를 Standby 로 대기시키기 위해 ReadOnly 로 변경하고

2. Master Active 에 붙지 않도록 connection 을 모두 종료시킨다. 

3. 그 다음 Master Active 에 신규 세션이 생기지 않도록 VIP 를 뺏는다.

4. Master Standby 와 Slave 를 비교하면서 복제 지연을 확인한다. 그 후 복제가 지연됐다면 Master Standby 기준으로
복제를 재구성한다. 

5. Master StandBy 를 Active 로 만들기 위해서 읽기모드를 제거해주고 VIP 를 붙여준다. 

- MMM 에서는 Master-Standby 가 항상 Failover 대상이 된다. 슬레이브가 아니라.

- FailOVer 이후에 이전의 Master Active 는 Master StandBy 로서의 역할을 하게 된다. 

#### MMM 에서 복제가 꺠지는 경우가 생길 수 있다. 

- Master StandBy 에서 Ack 를 날리는게 아니라 Slave 에서만 Ack 를 날리고 Master Active 가 장애가 나면
데이터 일관성이 맞지 않는다.  

- 이런 상황속에서 페일오버 대상이 Master Slave 기준으로 복제를 재구성하게 되고 이전에 장애가 생겼던 Master 가 
다시 서비스를 재개하게 되면 잃어버렸던 데이터가 다시 Master 에게 양방향 복제가 되고 그러면 Slave 에도 데이터 전송이 되게된다.
그러면 데이터 중복이 되고 복제가 꺠지게 되는 문제가 생긴다.

- MMM Failover 에서는 Slave 가 있는 경우에 데이터 복제 Crash 가 일어날 수 있다. 


#### MMM 개선 -> MHA(Master High Availability) 

MMM 과 마찬가지로 Perl 기반의 Auto Failover open source 프로젝트 이며 MHA Monitor 에서 
Master DB 서버의 Health Check 및 Failover 를 관리한다. 

MHA 는 Master + Slave 구조로 데이터는 모두 단방향으로 복제한다. 

MHA 와 MMM 의 페일오버의의 차이는 Master 에서 장애가 발생하게 되면 마스터는 데이터를 올바르게 가지고 있지 않다고
판단하게 되고 Slave 기준으로 복제를 재구성하고 Slave 중 최신 데이터를 가지고 있는 걸 마스터로 올린다. 

- 이 과정에서는 Binary Log 과 Relay Log 를 확인하고 비교해서 차이나는 이벤트를 추출하고 적용시킨다.

### 이중화 운영 장애

#### MMM 에서 네트워크 전면 장애가 난 경우

데이터베이스 장애가 일어난게 아니라 MMM Monitor 에서 Master 서버 접속이 불가능했다. 

이 경우 MMM Monitor 에서 FailOver 를 하는데 페일오버 과정중에 신규 마스터에 VIP 를 할당해야 하는데 접속이
되지 않으므로 VIP 를 붙일 수 없게 된다. 

이 장애를 극복하기 위해서 NHN 에서는 Secondary Check 를 이용했다. 페일오버가 작동하기 전에 Secondary Host 에서
접속이 되는지 확인하는 과정을 맡는다.   


### DNS 와 VIP 비교 

VIP 같은 경우는 Broad Cast 도메인 내에서만 구성이 가능하기 때문에 다른 지역에서 이중화가 불가능하다.

이를 해결하기 위해 DNS 를 이용했다. 
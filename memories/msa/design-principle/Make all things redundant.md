# Make all things redundant

https://learn.microsoft.com/en-us/azure/architecture/guide/design-principles/redundancy

***

- Build redundancy into your application, to avoid having single points of failure
- 이중화를 고려하라. 
  - fail over 를 가능하게 설계했는가? 

## Recommendations

- Consider business requirements
  - failover 와 failback 을 구현하는 건 비용이 든다. 프로세스 구축도 복잡하고. 
    - 그럼에도 불구하고 이걸 할 가치가 있는지 비즈니스적 요구사항을 확인해봐야한다. 이 뜻. 
  - failback 이라는 용어가 나왔는데, 장애가 나기 전 서버/시스템/네트워크로 되돌리는 처리를 말한다. 
    - failover 와 같이 쓰이는 용어이며 failover 는 장애가 났을 때 자동으로 다른 운영장비로 전환되는 기능. 

- Place VMs behind a load balancer.
  - load balancer 뒤에 하나의 장비 (= vm) 만 두지마라. 특히 비즈니스에 중요한 workload 라면. 
  - 여분을 준비하고 있어라 그런 뜻. 
  - 상태가 없는 장비에서의 이중화. 

- Replicate databases
  - 상태가 있는 장비에서의 이중화 

- Enable geo-replication
  - 지역별로 replication 을 준비해놔라 이 뜻. 

- Partition for availability
  - Database partitioning 은 scalability 를 줄 수 있으면서 가용성을 줄 수 있다. 
  - 하나의 샤드의 실패는 오로지 일부의 작업만을 방해하기 떄문에. 
    - **장애나도 움직이는 것. fault tolerance**

- Deploy to more than one region
  - 가용성을 위해서 하나 이상의 region 에 배포하는게 좋다. 
  - region 전체에 영향을 미치는 사건이 생길수도 있기 때문에. 
  - Multi-region application 으로 가야한다.

- Synchronize front and backend failover.
  - failover 에서 frontend 와 backend 를 동기화 시키는 것. 
  - 하나의 region 에서 frontend 에 도달하지 못한다면 failover 될텐데, 이때 backend 도 같이 전환되도록. 
  - Azure Traffic Manager 를 쓴다면 이게 가능하다. 

- Use automatic failover but manual failback 
  - Azure Traffic manager 를 쓰면 자동화된 failover 와 수동화된 failback 이 가능하다.
  - 자동화된 failback 은 리스크가 있다는듯. 완벽하게 healthy 하기 전에 전한될 가능성이 있어서. 
  - 그래서 verify 할 수 있는 단계를 넣어두라는 뜻. (이게 manual)
    - 데이터베이스 같은 경우도 데이터 일관성을 체크할 수 있도록. 
  - 모든 서브 시스템이 healthy 한 지 알수있는 `health endpoint monitoring pattern` 을 적용하라.

- Include redundancy for Traffic Manager
  - Traffic Manager 도 실패할 수 있는 포인트 중 하나다. 
  - 이거 하나만 쓰는게 맞는지 검토해봐라. 비즈니스 요구사항에 맞게. 
    - **하나 더 추가하는 비용이랑 장애가 났을 때 생기는 비용을 비교해봐라.**
      - 이것도 중요한 아이디어.

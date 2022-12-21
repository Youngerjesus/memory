## Throttling Pattern

https://learn.microsoft.com/en-us/azure/architecture/patterns/throttling

- (throttle 이 목을 조르다 라는 뜻도 있지만 자동차 조절판이라는 뜻도 있음.)
- application 이 consume 하는 속도를 제어하는 것.
- service-level 의 합의점을 찾도록 하는 것. 대규모 부하가 오더라도.
- 시스템의 요구사항을 초과하는 부하를 만나면 performance 는 떨어지고 실패할 것.
- 부하에 대처하는 전략은 많다. 그 중 하나가 auto scaling.
    - 근데 autoscaling 도 즉시 일어나는게 아니다. 일어나기 전까지는 위험할 수 있음.
- 이 전략은 제한된 범위까지만 사용할 수 있도록 제한 시키는 것.
    - 시스템은 리소스를 모니터링 하다가 지나치게 사용한다면 요청을 throttle 하는 것.
        - 요청을 거절하는 것.

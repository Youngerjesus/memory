##  Bulk Head Pattern

https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead

- tolerant of failure 를 위한 것.
- Application element 는 pool 로 isolate 되기 떄문에 하나가 실패해도 다른 녀석들은 계속해서 동작이 가능하다.
  배로 비유하자면 선체가 훼손되면 그 부분만 물로 채워지도록 해서 배 전체가 잠기는 걸 막는다.

- 부하가 강해서 전체가 다운될 수 있는 데 그걸 부분 실패로 막도록 하는 것인가
- Client 의 요청이 응답오지 않는다면 그만큼 블라킹 당하는 상황 떄문에 장애로 이어진다는 것.
    - 그치 커넥션 풀이 소진되는 것.
    - 그래서 요청을 못보내는 상태가 됨.

- 해결책은 partition 을 나누는 것. 하나의 서비스에 대한 호출이 다른 서비스 호출에 영향을 주지 않도록. 각 서비스마다 커넥션 풀을 두는 것.
  Circuit breaker 와 throttling pattern 도 같이 적용해보면 좋다.

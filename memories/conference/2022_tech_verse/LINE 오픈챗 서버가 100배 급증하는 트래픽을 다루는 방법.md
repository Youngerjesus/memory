# LINE 오픈챗 서버가 100배 급증하는 트래픽을 다루는 방법 -한국어판-

https://www.youtube.com/watch?v=L5ZjChLvd7U

***

- 골고루 많은 부하가 아니라 특정 하나의 부하가 몰리는 경우.

- 하나의 인기있는 hot chat 때문에 발생한 문제
  - 검색이랑 많이 비슷하다고 생각함.
  - 하나의 인기있는 검색어가 있는 것처럼.

- 오픈챗 지표
  - 하루에 100만개 요청
  - 하나의 오픈챗은 수만명까지 있을 수 있음.
  - 하나의 활발한 오픈챗은 1분에 20만개의 요청도 생긴다.

- 오픈챗 서버는 오픈챗의 모든 활동을 이벤트로 간주

- chatId 를 기반으로 한 샤딩. 더 많은 채팅방을 늘릴 수 있음.

- 하나의 인기있는 오픈챗을 위해서 전체의 Replication 을 늘리는 것은 오버헤드.

- Hot chat 을 탐지하고 여기서 Fetch event 의 수를 줄이는 방법을 생각해보자. 
  - 이걸 Hot chat detection & throttling 이라고 한다.
  - Hot chat detection
    - Kafka consume 타이밍에서 api 수를 비교한다. Threshold 를 정해놓고.
    - Line 의 central dogma 를 이용함.
    - 서버의 재가동 없이 동적으로 변경이 가능함.  

  - Throttling
    - 서버 푸쉬를 보내기 전에 핫 챗인지 확인
    - 핫챗이라면 서버 푸쉬를 확률적으로 throttling 한다.
    - Throttling 의 뜻은 과열되기 전에 클럭과 전압을 강제적으로 낮추거나 전원을 꺼서 발열을 줄이는 기능.

    - 대량의 fetch event 를 줄이는 방법인데, 그러면 유저가 이벤트를 받을 수 있나?
      - 유저는 한 개의 이벤트만 받으면 된다고함. 그럼 다 가져갈 수 있으니
      - 이벤트마다 모든 걸 fetch 를 하는 구조였던듯.
      - 그래서 못받더라도 하나만 이벤트를 가져가면 되는 구조.

    - 요청을 견디도록 하는 방식보다, 요청을 안정적으로 처리하는 방법에 대한 것. 
      - 부하가 와서 다운되는 문제 해결.
      - 빠른 실패로 인해 다시 요청하고 응답을 받는 구조.
      - Slow Query 와 타임아웃이 사라짐.
      
- Join spikes on hot chat
  - 1초에 2천개의 join 이 생겼다.
  - Improve bottleneck on MySQL
    - Insert chat member query 를 개선
    - WHERE 절에 Exist 로 subQuery 를 날리면 InnoDB AUTO_INCREMENT Lock Modes 가 Bulk Inserts 모드가 된다고함.

  - InnoDB AUTO_INCREMENT Lock Modes
    - Simple Inserts
      - Single row, multiple row insert replace
      - 내부에 Subquery 가 없어야함.
      - INSERT … ON DUPLICATE KEY UPDATE 도 아니어야함.
    - Bulk Inserts
      - INSERT … SELECT, REPLACE … SELECT, LOAD DATA 문인 경우에 해당
      - innodb_autoinc_lock_mode = 1 와 bulk insert 인 경우에는 table-level 의 lock 을 가진다.
  - `innodb_autoinc_lock_mode` 이 값을 2로 바꿔줘서 테이블 락이 안걸리도록 변경함. 


- MySQL Query Cache
  - SELECT 쿼리의 결과를 캐싱해주는 query cache 라는 기능이 있다.
  - MySQL 5.7.20 부터는 deprecated 되었고, 8.0 부터는 제거되었음.
  - 이미 캐싱된 query 라면 parsing, optimizing, executing 을 하지 않는다 라는 장점이 있다.
  - 대신에 만약 테이블에 대한 변경 (Insert, Update, Delete) 가 있다면 기존의 캐시를 제거한다는 점
  - 쿼리 캐시는 공유 자원이라서 이 경우 lock 이 걸리고 유효해질 때가지 대기해야한다.
  - 이 lock 을 query-cache-lock 이라고한다.


- Bulk Head Pattern
  - tolerant of failure 를 위한 것.
  - Application element 는 pool 로 isolate 되기 떄문에 하나가 실패해도 다른 녀석들은 계속해서 동작이 가능하다.
배로 비유하자면 선체가 훼손되면 그 부분만 물로 채워지도록 해서 배 전체가 잠기는 걸 막는다.

  - 부하가 강해서 전체가 다운될 수 있는 데 그걸 부분 실패로 막도록 하는 것인가
  - Client 의 요청이 응답오지 않는다면 그만큼 블라킹 당하는 상황 떄문에 장애로 이어진다는 것.  
    - 그치 커넥션 풀이 소진되는 것.
    - 그래서 요청을 못보내는 상태가 됨.

  - 해결책은 partition 을 나누는 것. 하나의 서비스에 대한 호출이 다른 서비스 호출에 영향을 주지 않도록. 각 서비스마다 커넥션 풀을 두는 것. 
  Circuit breaker 와 throttling pattern 도 같이 적용해보면 좋다.


- Throttling
  - 입력 주기를 방해하지 않음. 일정 시간 동안 입력을 모아서 출력을 제한한다.
  - API Gateway 의 경우 버스트 한도를 초과할 경우 429 (Too Many Requests) 에러를 던진다.
  - 이 버스트 한도를 아는게 중요하다.
  - 그리고 클라이언트는 이 에러를 보고 속도를 제한해서 실패한 요청을 다시 제출하도록 한다.
  - 톰캣의 maxThredCount 의 개수에 따라서 스로톨링 설정을 하면 되지 않을까.

- Throttling Pattern 
  - (throttle 이 목을 조르다 라는 뜻도 있지만 자동차 조절판이라는 뜻도 있음.)
  - application 이 consume 하는 속도를 제어하는 것. 
  - service-level 의 합의점을 찾도록 하는 것. 대규모 부하가 오더라도. 
  - 시스템의 요구사항을 초과하는 부하를 만나면 performance 는 떨어지고 실패할 것. 
  - 부하에 대처하는 전략은 많다. 그 중 하나가 auto scaling.
    - 근데 autoscaling 도 즉시 일어나는게 아니다. 일어나기 전까지는 위험할 수 있음. 
  - 이 전략은 제한된 범위까지만 사용할 수 있도록 제한 시키는 것.
    - 시스템은 리소스를 모니터링 하다가 지나치게 사용한다면 요청을 throttle 하는 것.
      - 요청을 거절하는 것.

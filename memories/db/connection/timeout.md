# 여러가지 타임아웃 

- DBCP (Database Connection Pool) 타임아웃 
  - 커넥션 풀에서 커넥션을 가지고 오기까지 대기시간을 말한다. 

- 데이터베이스 커넥션에서는 여러가지 타임아웃이 있다. 
  - 가장 상위인 스프링과 같은 프레임워크에서 지정하는 트랜잭션 타임아웃.
  - 그 다음은 쿼리를 실행하는데 지정하는 타임아웃인 Statement 에서 지정한 타임아웃 (Slow Query 방지.)
  - JDBC 드라이버 수준의 소켓 레벨에서 지정하는 타임아웃
  - 운영체제 레벨의 소켓 타임아웃 

- 상위 타임아웃은 하위 타임아웃에 의존한다. 소켓 레벨에서 지정하는 네트워크 읽기/쓰기의 타임아웃이 나지 않고 블라킹된다면 상위 레벨의 타임아웃인 Statement 는 쓰이지 않는다. 
  - 대신 소켓레벨보다 하위인 운영체제에서 지정한 타임아웃이 쓰인다.
  - 아마 TCP KeepAlive 와 같은 타임아웃이 쓰이겠지. (대략 30분.)
  - 즉 Statement Timeout 은 Network 타임아웃을 탐지할 수 없다. (transaction timeout 은 설정되면 특정 시간이 지난 후 예외를 던진다.)

- Statement Timeout 이 Socket Timeout 에 의존하는 이유는 어떤 동작 원리 떄문일까? 
  - 커넥션을 맺어서 쿼리를 보내는 동작 과정은 이렇다.
    - 1) Connection.createStatement() 를 통해서 Statement 를 생성한다.
    - 2) Statement.executeQuery() 를 호출한다. 
    - 3) Statement 는 내부 커넥션을 통해서 쿼리를 DBMS 에 보낸다.
    - 4) Statement 는 타임아웃 처리를 위해서 동작한다. (MySQL 은 타임아웃 처리용 스레드를 만들어서 처리하고 SQLServer 는 TimerThread 에 Statement 를 등록한다.)
    - 5) 타임아웃이 발생한다.
    - 6) 타임아웃 처리 스레드가 Statement 와 동일한 설정의 커넥션을 생성해서 취소 쿼리 (KILL QUERY "connectionId") 를 보낸다.
  
  - 타임아웃 처리를 등록하기 전에 DBMS 가 장애가 나서 read 를 못하면 이를 계산하지 못해서 그런게 아닐까? 
    - 맞다. 쿼리 실행문을 보내고 result 가 와야지 elapsed time 을 계산할 수 있으니까.  

  - Socket Timeout 이 지정되지 않은 상태에서 네트워크 장애가 생기면 무제한 대기하게 된다. 

- Socket Timeout 이 너무 짧으면 이거에 맞춰서 실패한다. 그래서 Statement Timeout 보다 커야한다.

- Transaction Timeout 의 경우에는 Statement Timout * N (= Statement 개수) + a 로 계산하면 된다.  


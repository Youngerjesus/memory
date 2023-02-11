# Statement Pool 튜닝 

- Statement Pool 이란 PreparedStatement 를 캐시로 가지고 있는 pool 이다. 
  - PreparedStatement 라는 건 SQL 문을 미리 컴파일해서 가지고 있어서 같은 SQL 문이라면 바로 보낼 수 있도록한다. 처리량을 높일 수 있다.

- Connection Pool 에서 Statement 를 체크한다. 이를 처리할 수 있는 SQL Statement 가 이미 존재하는지 보는 것.
  - Connection 마다 Statement Pool 이 쓰인다. 그래서 메모리에 부담을 줄 수 있다. 그게 단점임. 

- 이 Statement Pool 개념은 Hibernate 에는 없다고 한다. 대신 자기만의 캐싱 기법이 있다고함. 데이터베이스와의 통신을 효율적으로 하기 위해서. 

- Statement Pool 에 있는 내용이 JVM Old generation 영역으로 가기만 한다면 매번 PreparedStatement Object 를 만들 필요가 없어서 성능이 더 나아질 것.
  - 여기서 중요한 아이디어는 HTTP Request 가 들어오고 나갈 때 GC 대상이 되도록 해서 매번 사라지고 다시 만드는 객체 보다는 그냥 Old 영역에서 보관하는게 더 성능이 나을 수 있다는 것.
  - 특히 캐시가 그럴 거 같다. 캐시는 적중률이 계속 높아야 Old 영역으로 갈 수 있기 떄문에. 
  - 이런 객체를 계속 가지고 있지 않고 제거되게 만든다면 Old 영역으로 넘어간 객체는 기존에 계속 보관되는 객체보다 제거되고 하면서 FullGC 에도 부담을 줄 수 있다.


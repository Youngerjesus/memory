# Bridge

***

구조적인 디자인 패턴

브릿지 패턴은 추상적인 것과 구체적인 것을 분리해서 연결하는 패턴이다.

- 하나의 계층 구조로 가는게 아니라, 각각 나눠서 독립적인 계층 구조로 만들어서 각각 발전을 시키는 것
- 서로 성격이 다른 것들을 나누고 연결한다는 뜻.
    - 성격이 다른 것들이 하나의 클래스에 모이면 겉잡을 수 없이 클래스가 커질 수 있기 때문에 사용한다.
- 클라이언트의 입장에서는 Abstrsction 을 바라보고 사용한다.
    - Abstraction 은 근간이 되는 추상적인 정보로 확장을 해나가는 걸 말한다.
    - Implementation 은 구체적인 정보로 객체의 상태, 행동 등을 별도의 계층구조로 나누는 것

### 장점

- 추상적인 코드를 구체적인 코드에 의존하지 않고 확장하는게 가능하다. (OCP (Open-closed principle), SRP 를 만족한다.)
-

### 단점

- 계층 구조가 늘어나서 복잡도가 증가할 수 있다.

### 실제 예

```java
try (Connection conn = DriveManager.getConnection(url, user, password)) {
	String sql = ~
	Statement statement = conn.createStatement(); 
	statement.execute(sql); 
}
```

- DriverManager 와 각 DB 와 연결하는 Connection 은 브릿지 패턴의 관계
    - MySQLDriverMangager, H2DriverManager 이렇게 사용하지 않는다.
    - 하나의 DrvierManager 와 각 Driver 들이 있고 이것들이 연결된 관계.
    - 이로인해 새로운 데이터베이스 Driver 가 나온다고 해서 DriverManager 가 변경되지 않는다. DriverManager 가 내부적으로 가지고 있는 registerDrivers 에 새로운 Driver 를 추가시켜놓으면 된다.(객체를 생성하는 시점에)
-
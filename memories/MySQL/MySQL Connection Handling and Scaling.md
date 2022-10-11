# MySQL Connection Handling and Scaling

https://dev.mysql.com/blog-archive/mysql-connection-handling-and-scaling/

## 다루는 내용

- MySQL Connection, User Threads, and Scaling

## MySQL Connection working

- MySQL Server (mysqld) 는 하나의 프로세스 그리고 여러개의 스레드로 동작한다.
- User 가 MySQL 에 connect 를 한 순간 mysqld 에는 user thread 가 생겨난다.
    - 이 User Thread 가 쿼리를 날리고 결과를 가져오고 하는 역할을 한다. (연결이 끊길 때까지.)
    - User Thread 가 많이 생겨나면 이것들을 병렬로 처리한다.
    - 즉 User Thread 가 어느 순간에 도달하면 더이상 처리량이 높아지지 않고 비효율적이됨.
- MySQL Client (command line tool or application) 과 mysqld 가 통신하는 프로토콜은 `libmysqlclient` 라이브러리를 사용한다.

## Connect and Disconnect

![](./images/mysql_connect.png)

- `Connection Request`
    - connection 요청은 TCP-IP 를 따른다.

- `Receiver Thread`
    - 들어오는 커넥션 요청은 큐에 쌓이게 되고 리시버 스레드에 의해서 하나씩 처리된다.
    - 리시버 스레드가 하는 일은 이 요청에 대해서 유저 스레드를 만드는 일이다.

- `Thread Cache`
    - 리시버 스레드는 OS 의 스레드를 새로 만들거나 만들어져있는 걸 재사용해서 유저 스레드로 만든다.
    - 재사용 하는 이유는 스레드 생성의 비용이 비싸서.
    - `thread_cache_size` 의 기본 값은 `8 + max_connections/100` 이다.
    - 만약에 커넥션의 개수가 많아질 수 있는 변동성이 있다면 이 기본 값을 늘려놓는게 좋다.

- `User Thread`
    - client-server protocol 을 담당한다. (packet 전달과 같은)
    - 밑에 나오는 THD 를 할당하고 초기화 하는 역할을 한다. (THD 가 쿼리를 실행하고 난 후 데이터를 가지고있는듯.)
    - THD 초기화 후 유저의 credential 을 바탕으로 인증 과정을 처리하고 통과하면 THD 의 Security Context 에 저장된다.
    - 이런 Connection Phase 가 통과되고 나면 Command Phase 로 들어간다. (command phase 에서 유저는 쿼리를 날릴 수 있다.)

- `THD`
    - Connection 이 표현되는 데이터 스트럭처는 THD 이다.
    - 커넥션이 만들어질 때 생기고 커넥션이 사라질 때 이 객체도 사라진다.
    - 즉 User Connection 과 1:1 관계.
    - 이건 재사용하지 않음.
    - THD 사이즈는 10K 이고 이는 `sql_class.h` 에서 확인이 가능함.
    - THD 를 통해서 `execution state` 를 추적하는게 가능함.
    - user query 에 따라서 THD root 의 메모리는 증가한다. (기본적으로 10MB 정도 쓰일 것이라고 MySQL 은 예상중)
    - 커넥션의 모든 쿼리는 같은 THD 에 의해서 다뤄진다.
    - 여기서 쿼리를 처리하는데 여러개의 statement 가 하나의 트랜잭션 내에서 처리되도록 원할 수도 있다. 그러므로 `transaction context` 를 지원한다.
    - `auto-commit mode` 에서는 자동으로 트랜잭션으로 실행된다.

### active connection mode

![](./images/active_mode.png)

### disconnect mode

![](./images/disconnect_mode.png)

- disconnect 할 때 클라이언트는 `COM_QUIT` 명령을 보낸다.
    - 이 명령을 통해서 소켓을 닫는다.
- disconnect 의 경우 Thread Cache 에 slot 이 있다면 거기로 돌아가고 돌아갈 slot 이 없다면 유저 스레드는 지워진다. 그리고 THD 객체도 지워진다.

## Short Lived Connections

- 짧은 커넥션을 유지하는 어플리케이션의 경우데도 MySQL 은 꽤 성능이 잘나온다.
    - 초당 80,000 connection/disconnection 까지는 괜찮게 성능이 나온다는 벤치마크 자료가 있었음.

## Long Lived Connections

- 커넥션을 무기한 연기하는 어플리케이션의 경우에는 connection 을 `max_connections` 개수만큼 받을 수 있음
    - 이거 넘어가면 못받음
- Connection 의 개수는 두 가지 요소에 좌우된다.
    - **클라이언트가 보내는 부하의 양**
    - **MySQL 서버가 점유하는 하드웨어 리소스양에 따라서 달려있다.**
    - 200 user 에서 5000 TPS 가 뽑힌다면 더 많은 Connection 을 가진다고 성능이 더 올라가지 않는다. 내려갈 일만 남음.
    - 최대 TPS 를 측정하는 방법은 유저의 수를 두 배씩 늘려보면서 하면 된다. TPS 가 꺾이는 순간이 올거임.
    - 벤치마크 자료를 보면 TPS 는 어떤 지점까지 계속해서 올라가고 꺾이고 응답속도도 특정 순간 이후로 폭팔적으로 증가한다.

## What Limits Thread Concurrency?

- Concurrency 를 제한하는 세 가지 요소가 있다.
    - Mutex
    - Database lock
    - I/O
- Mutex
    - 공유자원에 접근하기 위한 티켓 같은 것
    - Mutex 를 통해 오로지 하나의 스레드만이 공유 자언에 접근할 수 있음을 보장함.
- Locks
    - MySQL 의 InnoDB 는 MVCC 를 통해 Lock 을 피하는데 유용하다.
    - MVCC 는 old 버전의 row 를 기억하는 것을 말한다. 락을 최대한 피하기 위해서.
    - 락은 크게 두 종류가 있다. DML 쪽 data lock 과 DDL 쪽 meta-data lock.
    - 락이 있으면 성능을 깎는 건 맞다.
    - MySQL 8.0 에서는 개발자에게 새로운 기능도 제공해준다.
        - NO_WAIT
            - 락이 걸려있다면 실패처리. 원래는 `innodb_lock_wait_timeout` 만큼 기다려야함.
        - SKIP_LOCKED
            - row 에 lock 이 걸려있다면 해당 row 는 skip 하고 리턴
- IO
    - 메모리에 없어서 I/O 를 자주한다면 그만큼 User Thread 는 놀게될 것.



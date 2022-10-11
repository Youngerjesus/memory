# Read Isolation, Consistency, and Recency

## 먼저 Read Concern 에 대해서

### local

- replica-set 의 대부분 멤버 (majority) 에 써져있다는 걸 보장하지 않는 데이터를 읽어올 수 있다. (즉 롤백될 수 있는 데이터임.)
    - 이게 기본값이다.
    - consistent sessions 과 transaction 에 사용할 수 있고 사용하지 않을 수 있다.
    - 트랜잭션에 사용하면 딱 트랜잭션을 위한 Read Concern 을 지정하는게 가능함.
    - 트랜잭션 안에서 Collection 과 index 를 만들 때는 `local` 만 지정하는게 가능하지만 그게 아니라 기본적인 DML 인 경우 (find, update, insert) 는 어떤 Read Concern 도 가능함.

### available

- 기본적으로 local 과 같음.
- 다만 샤드 클러스에서 orphan document 를 조회할 가능성도 있음.
    - 왜냐하면 샤드 config server 정보 업데이트를 기다리지 않기 떄문에.


### majority

- replica-set 의 대부분 멤버에 쓰여진 것들만 읽는 것.

### linearizable

- **read operation 을 수행하기 전에 write 의 작업이 대부분 replica-set 의 멤버에 쓰였는지 확인한 후 결과를 가져온다. (기다린다.)**

## Isolation Guarantees

- isolation 은 operation 이 별개로 운영되는 상황에서 어떻게 읽는지를 말하는듯.

### Read Uncommitted

- 다소 오해를 했었는데 Read Uncommitted 은 커밋될 떄까지 기다리지 않음. 그냥 읽음. (Read Concern 의 local 수준.)
    - 샤드 클러스터 환경에서 멀티 도큐먼트를 업데이트 하는 경우에 하나의 문서는 커밋되었지만 다른 문서가 커밋되지 않았다면 local 은 커밋된 문서까지만 읽음. 즉 기다리지 않음.
    - `Multiple Documnet Atomicty` 상황.
- `Single Document Atomicity` (하나의 문서를 업데이트 중인 상황에서 읽는 상황) 업데이트 된 문서를 볼 수 없다. 하지만 `Read Uncommitted` 에서는 볼 수 있다.
    - 이건 Single Document 에서의 업데이트는 원자성을 가지고 있기 때문에.
    - **standalone 이나 replica-set 의 경우 하나의 문서에 대한 읽기, 쓰기 명령은 serializable 이 가능하다. 다만 replica-set 은 롤백이 안될때만 가능함. (롤백이 안될 떄는 정상적인 케이스를 말하나, 승격하면서 롤백될 수 있으니까.)**
- `Multiple Document Atomicty` 에서 하나의 문서 갱신은 원자적이지만 여러개의 문서 갱신은 원자적이 아니므로 중간에 끼어들어서 읽는게 가능함. 모든 연산을 원자적으로 읽을려면 트랜잭션을 써야함.
    - 즉 100개의 데이터를 넣는 상황에서 중간에 쓰여진 데이터만 읽는것도 가능하다는 뜻.

### Without isolating the multi-document write operations, MongoDB exhibits the following behavior:

- 트랜잭션을 쓰지 않는 상황이라면 금지해야 할 것들을 정리하는 것.
- `Non-point-in-time read operations`
    - 데이터의 특정 시점 스냅샷을 보는게 불가능하다는 뜻. (업데이트 되고 있을 거니까.)
    - `t1` 일 때 문서들을 읽기 시작하는데 `t2` 에서 그 문서들 중 하나를 업데이트 한다면 업데이트 된 문서가 끼어있을 수 있다.
- `Non-serializable operations.`
    - read-write, write-read 연산 의존성이 생긴 경우. 각 경우마다 연산을 직렬화해서 수행하는데 read-write, write-read 가 순환적으로 이뤄지면서 직렬화를 할 수 없어진다.
        - **read-write 는 스냅샷을 보는거고, write-read 는 업데이트 된 것을 보는 것이라고 생각해보자.**
        - **근데 순환 의존성이 발생하면서 이게 가능해지지 않는다라는 뜻.**
    - `serializable schedule` 에서 이게 안되는듯. 읽고 쓰기가 얽힌다고만 알자.
- 업데이트 되고 있는 문서에서 읽기가 실패할 수 있따.

## Cursor Snapshot

- cursor 를 열어놓으면 계속해서 문서를 참조할 수 있는데 이 중에 인덱스가 바뀐다던지 하면 같은 문서를 또 참조할 수 있다.

### Real Time Order

- Read Concern 을 `linearizable` 그리고 Write Concern 을 `majority` 로 지정해놓으면 마치 하나의 스레드가 데이터를 변경하고 읽기를 가져오는 것 같은 효과를 내놓을 수 있다.
- 즉 변경을 하면 바로 반영됨. read 가 write 를 기다리니까.

### Causal Consistency

- 이전에 설명한 적이 있으므로 생략.
- 쿼리 간의 의존성이 있을 때 사용가능 

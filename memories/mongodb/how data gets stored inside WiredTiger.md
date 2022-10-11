# how data gets stored inside WiredTiger

http://source.wiredtiger.com/mongodb-3.4/tune_page_size_and_comp.html

***

## Data life cycle

- WiredTiger 는 실제 디스크에 데이터를 저장하고, database directory 에 on-disk file 을 테이블을 위해서 만든다. 그리고 이 데이터들을 메모리에 캐싱해놓는다. read 와 write 작업을 위해서.
- 캐싱을 해놓을 때 메모리에서 데이터 구조는 B-Tree (정확하겐 B+Tree) 를 이용한다. (B-Tree 는 페이지 참조를 이용하는 방식.)
- 메모리에 있는 데이터 포맷과 디스크에 저장하는 포맷은 다르기 떄문에 디스크에 저장할 떈 디스크 구조로 만들어줘야한다. 이 과정을 reconciliation 이라고 한다. (메모리 페이지 구조를 in-memory page, 디스크 페이지 구조를 on-disk-page 라고 부름.)
    - on-disk page 의 최대 사이즈는 어플리케이션에서 설정할 수 있고 그렇지 않으면 기본 값을 사용함.
    - in-memory page 의 reconciliation 과정 이후에 on-disk page 의 크기가 이 최대값보다 크다면 이 페이지는 쪼개져서 디스크에 저장된다.
        - 이 역할은 WiredTiger 에 있는 Block Manager 가 한다. (쪼개고 저장하는 것까지.)
- 일반적으로 디스크가 메모리보다 크기 떄문에 모든 데이터는 메모리에 올라올 수 없다. 그렇기 떄문에 eviction process 를 돌려서 새로운 데이터가 메모리에 할당할 수 있도록 자주 접근하지 않는 데이터는 할당 해제한다.
    - eviction server 가 제거해야 할 in-memory page 를 찾는 역할을 한다. (이건 LRU 알고리즘을 이용함.)
    - eviction background thread 가 여기서 찾은 페이지를 제거하는 역할을 한다.
- 어플리케이션에서 insert or update 작업을 <key, value> 를 통해서 하는 경우, <key> 를 통해서 in-memory page 에서 찾는다. 없다면 on-disk page 에서 찾고 in-memory page 형식으로 데이터를 바꿔서 반영한다.
    - 이떄 in-memory page 는 점점 커질 수 있다.
    - 이게 점점 커져서 in-memory maximum page size 에 도달한다면 이 in-memory page 도 쪼개진다.
    - (디스크에 있다면 메모리로 가져와야 한단는 소리네.)
    - 그리고 어플리케이션에서 insert or update 한 후 in-memory page 가 maximum size 에 도달한다면 어플리케이션 스레드는 이 page 를 evict 한다.
        - in-memory 페이지가 max 에 도달하면 이게 쪼개지고 on-disk page 로 변경해서 저장한다. 그리고 이후에 이 페이지는 메모리에서 삭제되는 것. 쪼개졌으니.
        - 이때 insert or update 는 꽤 시간이 걸릴 수 있다.
        - 일시적으로 in-memory page 가 maximum 에 도달할 수 도 있으니 바로 evict 되지는 않고 mark 만 한다. 그리고 더 많은 데이터를 넣을 때 재시도함. 


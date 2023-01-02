# 대량의 광고데이터 필터 기능 개발기 ES 한테 뺨맞고 Citus 한테 안긴 썰 

## ES 에셔 연관관계를 사용하기 위한 방법들

### Denormalization

- 중복되는 데이터를 넣는 것.
- Update 해야하는 document 수가 많아지고 Indexing 비용 증가.  

### Nested

- 오브젝트들을 하나의 문서에 다 담는 것.
- Nested 요소들을 통해서 검색하도록 할 수 있다.
- 쓰기 증폭이 생길 수 있다.
- 일부만 변경, 추가 되더라도 전체 document 를 다시 indexing 해야한다.

### Parent Join
- 특정 index 내에 연관 관계를 선언하는 것.
- 연관 관계와 무관하게 업데이트 가능하다.
- Multiple level 의 연관관계를 가지는 것을 추천하지 않는다. 각각의 관계는 쿼리 타임에 오버헤드를 부르기 때문에.
  - 이 오버헤드에 대해서 좀 자세하게 알아보고 싶다.
  - 관계를 연결시켜서 찾는게 오버헤드라고만 하는듯.
- Join 은 global ordinal 을 트리거링 한다.  
  - Global ordinal 은 join query 의 속도를 빠르게 하기 위한 것.
  - 샤드에 parent 가 많아질수록 이 global ordinal 을 만드는 rebuild 과정은 더 길어진다.
  - 이걸 안하면 첫번째 쿼리의 경우에 latency 가 엄청 높아질 것.
- Join field 는 RDBMS 의 join 처럼 사용해선 안된다.
- 올바르게 사용하는 경우는 products 와 offer 의 관계.
  - Product 와 offer 는 one to many 관계이면서 offer 는 product 보다 앞도적으로 많다.
- 인덱스당 join 매핑은 하나만 가능하다.
- 하나의 parent 에 여러개의 child 도 가능하다.
- Parent 와 child document 들은 모두 같은 shard 에 있어야한다.
- Join 선언 방법과 이용 방법에 대한 예제도 있으면 좋을듯.
  - Parent 를 색인할 떈 자기가 parent 라는 걸 구별해줘야한다.
  - Child 를 색인할 땐 parent 의 id 가 있어야 한다. 그리고 child 라는 것도 함께.
- Searching with parent-join
  - Parent-join 은 하나의 필드를 더 만든다. The name of the relation.
  - 내가 parent 인지, child 인지 구별해주는 것.
  - Per parent/child 간의 하나의 필드를 더 만든다.
  - Join 필드라는 이름이다.
  - Join field 와 # 그리고 parent 이름 이렇게 따라온다.
  - Child 문서라면 parent id 필드도 필요하다.

## ES 에서 저장하는 최대 데이터 규모를 측정

- 계정당 캠피인이 최대 1000 개
- 캠페인 1개당 광고그룹은 최대 1000개
- 광고그룹 1개당 키워드 최대 1000개
- 광고그룹 1개당 소재 최대 20개
- 키워드 보고서는 최대 2년치
- 정리하면 계정 1개당 2년 * 365 (일단위) + 24 (당일 시간 단위 데이터) = 754
  - 754 * 1000 * 1000 * 1000 하면 7500억개
  - 계정 2개만 되어도 1조개가 넘어갈 수 있음.
- 이렇게 계산하는 방법도 있고, 현재 사용량이 있다면 1.5배 곱해서 계산해서 스펙을 산정.


## ES 의 집계 기능
- 최대 bucket size (65,536) 개 만큼만 데이터 집계가 가능하다.
- 여기서는 최소 1000만개의 집계 기능이 필요하다고 함.
- 이 값을 10만으로 변경시키고 쿼리 날려봤는데 데이터 노드가 죽음


## Citus 소개
- postegresql 의 확장해서 Distributed database 로 사용이 가능하다.
- RDB 에서 샤딩 기능을 편하게 제공해주는 듯.
- 사용자별로 데이터 개수가 천차만별인데 쿼리 실행도 이거에 맞춰서 달라지려나 검토.
  - 500 만개 이상이라면 Full Scan
    - 전체 데이터 노드에서 쿼리 실행해서 coordinator node 에 반환.
  - 500 만개 이하라면 index Scan
    - Index 에 해당하는 노드만 data node 에서 실행하고 coordinator 노드에게 반환.
- 적합한 데이터베이스인지 확인하는 과정을 엄격하게 거쳤다고 생각함.
- 1) 광고 도메인 필드값 포함 또는 제외
   - 키워드 검색이 가능해야함.
- 2) 광고 도메인 필드값 일치하는지
   - 광고 도메인의 계층 구조를 고려한 상태값이 일치하는지
   - 하나의 필드에 선택된 옵션들은 OR 조건으로 합쳐서 필터링
- 3) 보고서 데이터 범위 질의
   - 최대 보관기간 2년

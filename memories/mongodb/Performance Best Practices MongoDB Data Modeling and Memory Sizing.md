# Performance Best Practices: MongoDB Data Modeling and Memory Sizing

https://www.mongodb.com/blog/post/performance-best-practices-mongodb-data-modeling-and-memory-sizing

***

## Data Modeling Matters

- application query pattern 을 먼저 생각해보는 것.
    - **너무 당연한 거라서 패쓰**

## Key Considerations and Resources for Data Modeling

- data relationship 을 생각해보는 것.
    - nesting 구조 or embed a document 구조로 갈 수도 있고, 다른 컬렉션에서 reference between separate documents 구조로도 갈 수 있다.
- **검색 결과 예시도 결국 쿼리 조회 후 문서 조회하는 구조인데 현재 구조면 두번씩 호출하는게 아깝긴하다. 근데 뭐 어쩌겠냐. 문서 사이즈 제한이 있는데 **

### Embedding

- 1:1 관계라면 하나의 문서안에 포함되야함.
- 1:N 이라도 부모 도큐먼트 맥락 속에서 보여지는 애라면 이것도 문서안에 포함되야함.
    - 왜냐하면 함께 쓰일거니까.
    - 이 경우 locality 가 좋다고 함. 더 나은 퍼포먼스를 낼 수 있음.
    - atomic 한 연산이 가능 .

### Referencing

- 1:1, 1:N 이더라도 참조 구조로 가는 경우
    - 접근하지 않는 데이터가 있다면.
    - 한 부분은 계속 커질 여지가 있지만 다른 부분은 정적이라면
    - 합쳐진 도큐먼트가 16MB 를 넘는다면.
        - 대표적으로 제품의 Review
- 다만 레퍼런스 구조로 가면 여러번 쿼리를 날려야하는데 network 비용이 생긴다. 이를 `$lookup` 을 쓰면 조인과 같은 오퍼레이션이 됨.

## Memory Sizing: Ensure your working set fits in RAM

- application working set 이 모두 메모리에 있을 때 몽고 DB 는 최고로 작동한다.
    - **캐시와 같네**
    - 다른 최적화는 영향이 없을 정도로 이게 중요하다는 뜻.
    - 다만 성능보다 가성비가 중요하다면, 모두 램으로 채울 수 없다면 SSD 를 써야한다. 

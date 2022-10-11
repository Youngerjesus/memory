# Performance Best Practices: Query Patterns and Profiling

https://www.mongodb.com/blog/post/performance-best-practices-query-patterns-and-profiling

***

## Start by using the latest drivers

- 드라이버는 database 보다 좀 자주 업데이트 되는 편. 몇달 주기로 된다는 듯.
    - 아무래도 최신 걸 쓰면 그만큼 더 개선이 된 점이 있지 않겠나 라는 것.

## Avoid creating large, unbounded documents

- 도큐먼트의 사이즈의 최대는 16MB 이니 이거보다 커지지 않도록, 한계없이 커지지 않도록 하는 걸 막아야한다.
- 예로 제품의 리뷰 같은 경우는 엄청 커질 수 있음. 근데 이런 리뷰들 중 가장 인기있는, 가장 도움이 많이 된 일부 리뷰들만 쓰이고 나머지 리뷰는 고객이 보고 싶어하지 않을 것.
- 그러므로 제품 도큐먼트에 일부 리뷰들의 소집함만 넣고 관리하고 나머지 리뷰는 또 다른 컬렉션에서 관리하는게 맞을듯.

## Issue updates to only modify fields that have changed

- 도큐먼트를 찾아서 update 날리는 것보다 그냥 업데이트 하고 싶은 구체적인 필드를 찾아서 날리라는 뜻임. 이게 더 효율적.

## Update multiple array elements in a single operation

- `arrayFilter` 와 같은 fully expressive array update 를 이용하면 도큐먼트 내부에 있는 어레이들에게 하나의 연산으로 업데이트 치는게 가능함.
    - 내가 업데이트 칠 어레이 내부에 있는 애들에게만 조건을 거는게 가능한듯.

## Profile queries with the explain plan

- explain 을 사용해서 쿼리가 어떻게 실행되는지 확인
    - 어떤 인덱스가 사용되는지
    - 인덱스가 사용되는지 아닌지
    - 메모리 내에서 정렬을 수행하는지 여기서 인덱스가 도움되는지
    - 인덱스 스캔은 얼마나했는지
    - 도큐먼트 스캔을 얼마나 했는지
    - 쿼리가 얼마나 걸렸는지
    - 쿼리 플랜에서 제거된건 뭔지 

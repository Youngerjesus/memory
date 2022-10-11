# Object Storage 구현을 위한 분산 Radix Tree

https://tech.kakao.com/2022/08/18/radix-tree-for-object-storage/

## LSMT(Log-Structured Merge Tree)와 Range Query

- HBase 와 Cassandra 는 range 쿼리를 위해서 LSMT (Log-Structured Merge Tree) 를 기반으로 한다.
    - range query: 상한과 하한 사이에 검색을 하는 쿼리
- LSMT 는 삽입된 데이터를 모아서 정렬하고 저장한다. 이렇게 정렬되서 저장된 테이블을 SSTable (Sorted Strings Table) 이라고 부르기도 한다.
- Range 쿼리를 하는 경우에 모든 SSTable 에 Binary Search 를 해야하는데 꽤 많은 CPU 와 I/O 를 동반한다.
    - 이 작업을 효율적으로 하려면 SSTable 을 하나로 합치는 Compaction 작업을 진행해야하는데 이것도 CPU 와 I/O 가 비싸다.

## 그렇다면 분산 BTree 는 어떄?

- LSMT 이전에는 BTree 를 사용했다.
- BTree 는 갱신이 있을 때 In-place Update 를 통해서 Tree 를 수정한다.
    - 일반적으로 BTree 의 경우에는 정렬되어 있으므로 Range Query 에도 좋다.
    - 그냥 Write 만 하는 거라면 LSMT 가 좋다. (LSMT 는 Sequential Write 라면 BTree 는 Random Write 니까.)
- 다만 `Web Scale` 의 환경에서는 Random Write 에 강한게 좋다.
    - `Web Scale` 은 대규모 환경에서 고품질의 서비스를 비즈니스 환경에 맞춰서 신속하게 제공해주는 패턴을 말한다.
- 다만 BTree 는 분산 환경에서 데이터 정합성에서 문제가 있을 수 있다.
    - BTree 의 갱신을 보면 다음과 같다.
    - 새로운 데이터를 넣을 때 데이터가 가득차서 `Split` 이 일어난다면 새 노드를 만들고, 새 노드로 기존 노드의 item 을 올리고, 부모 노드에 새 노드의 링크를 추가하는 식으로 이뤄진다.
    - 이 세 가지 과정이 모두 성공해야 한다. 하나라도 실패하면 안된다.
    - 하지만 분산 환경에서 Transaction 은 비싸다. 그래서 BTree 는 적합하지 않음.
    - 또 BTree 는 루트 노드에 대한 병목 현상이 있다.

## 새로운 해결 방법 Radix Tree

Radix Tree 는 매우 큰 용량을 가진 Hash Table 위에 구축된다.

![](./images/radix_tree_logical_structure.png)

- 구조를 보면 노드는 key-value 로 저장된다.
- 각 노드에 값들은 정렬되어 있다. Array 형태로.
- 모든 Node 의 Key 는 Node 의 Prefix 로 표현되고 `$` 라는 시스템 문자를 접두사로 가진다.
- 그리고 Node 는 Child 를 가지고 있고 자신의 Key + Child 를 합친 이름으로 Child Node 를 찾을 수 있다. 아이템이 Data Flag 를 가지고 있다면 실제로 Item 을 확인할 수 있다.

스캔 과정을 보자.

![](./images/radix_tree_scan.png)

1) `$` 문자를 바탕으로 root node 를 찾는다.

2) 루트 노드에서 binary search 로 `/1` 을 찾는다.

3) Node 의 Key 와 Child 를 합쳐서 Child Node `$/1` 을 찾는다.

4) Child Node 에서 바이너리 서치로 2345 를 찾는다.

5) Node 의 Key 와 Child 를 합쳐서 child Node `$/12345` 를 찾는다.

6) Node 에서 바이너리 서치로 Begin Range 인 `/89` 를 찾고 그 뒤의 child 를 찾는다.

Radix Tree 의 물리적 배치를 보자.

![](./images/radix_tree_physical_structure.png)

- 각 노드는 데이터 노드에 샤딩된다.

## Radix Tree 의 Split


![](./images/radix_tree_split.png)

- Radix Tree 는 삽입을 할 때 노드에 공간이 부족한 경우에 Split 과정이 생긴다.
- 새로운 Child Node 를 생성하고 거기에 일부를 보낸다.
- 이후에 옮겨간 데이터는 삭제하고 새 노드를 가리키는 값을 추가한다.
- 즉 정리하자면 두 번의 PUT 연산이 있는건데 두 번째 연산은 첫 번째 연산이 실패하면 동작하지 않고, 첫 번째 연산이 실패하더라도 큰 문제가 일어나지 않는다.
    - 왜냐하면 링크가 만들어지지 않았기 때문에 정합성 문제가 생기지 않기 때문이다.
- 추후에 cursor 를 통해 끊겨진 노드를 확인하고 Child Node 를 삭제하거나 복구하는 로직이 동작하게 할 수 있다.

## Bloom Filter 로 병목 제거하기

- Radix Tree 도 BTree 와 같이 루트 노드에 병목이 있다는 점은 동일하다.
    - 상위 노드 다 포함.
- Bloom Filter 를 통해 `Tree Traverse` 를 도와주는 것도 가능하다.
    - 이는 Node 의 존재 여부를 표현하는 Hint 다.
    - 이건 분산 Radix Tree 모듈의 메모리에 올라가있고 해당 노드가 있는지 확인해주는 용도로 쓴다.
    - 즉 중간 노드에서 부터 탐색하는게 가능해진다. 


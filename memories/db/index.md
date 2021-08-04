# 인덱스(Index)

***

## 클러스터링 인덱스 vs Non 클러스터링 인덱스 

#### 클러스터링 인덱스는 

- PK 와 유사하고 순서대로 데이터가 정렬되어있다. 그러므로 범위 검색에 아주 유용하고

- Auto Increment 가 아닌 존재하는 PK 사이에 INSERT 할 경우 데이터를 밀어내는데 이 경우 문재가 생긴다.

#### Non 클러스터링 인덱스

- 순서와 상관이 없다. 대신에 데이터를 찾을 주소와 연결이된다. 그리고 이것들은 테이블과 별도로 추가적인 인덱스를 넣을 수 있는 공간이 필요하다. 

- 하지만 테이블에 여러개가 존재할 수 있고 

- 카디널리티를 신경써야 한다. 

***

## 인덱스가 사용될 조건은? 

Where 절에서 `<, <=, >, >=, = , Between, In ` 같은 절을 사용하거나 AND 로 묶이는 경우에 사용이 가능하다.

OR 절 같은 경우는 싱글 테이블인 경우 Index Merge Scan 이 발생하는게 가능하다. 

*** 

## LIKE 조건인 경우에 인덱스 사용은? 

`%string%` 과 같이 와일드카드가 앞과 뒤에 사용한 경우 문자열 검색인 경우 글자가 3개 이상이라면 
Boyer-Moore 알고리즘에 따라 문자열 검색을 시작한다.    

***

## Hash Index vs B-Tree Index

해시 테이블 기반의 인덱스가 더 조회는 빠르다. HashTable 같은 경우는 조회에서 O(1) 이 나오지만 B-Tree 같은 경우는 O(logN) 이 나오기 떄문에.

그렇다고 무조건 해시 기반의 인덱스가 좋냐라고 물으면 그건 아닌데 HashTable 같은 경우가 메모리에 좀 더 맞춰진 자료구조이기도 하다. 

메모리는 Random Access 할 때 성능의 차이는 없지만 디스크는 Random Access 접근을 할 때 linear access 보다 성능이 더 안나오기도 하기 때문이기도 하고

그리고 해시 테이블 기반의 인덱스는 range query 가 되지 않는다.  

그리고 Hash Table 구조의 인덱스의 경우 >, >= 같은 연산이 가능하지만 <, > 같은 비교는 되지 않는다. 
***


## Column Indexes

하나의 칼럼은 읻덱스로 사용하는게 가능하다. B-Tree 기반의 자료구조를 사용하므로 범위를 좁혀주는 연산일 때 사용이 가능하다. 

예를 들면 (=, >, <=, between, 와일드 카드로 시작하지 않는 LIKE)

Column Indexes 의 종류는 다음과 같다. 

- Index Prefix

- FULLTEXT Indexes

- Spatial Indexes

- Indexes in the MEMORY Storage Engine 

#### Index Prefixes

string 칼럼에서 앞에 N 개의 문자를 이용해서 인덱스로 사용하는 걸 말한다. 주로 BLOB 나 TEXT VARCHAR 칼럼에서 사용한다. 


#### FULLTEXT Indexes

FULLTEXT 인덱스는 말 그대로 full-text search 에 사용한다. 작동 동작은 Full-Text Search Function 에 의해서 작동하는데

이는 나중에 따로 설명하겠다. 

#### Spatial Indexes

주로 Point 나 Geometry 같은 Spatial data type 에 사용하는 인덱스다.

***

## 인덱스 조건

- WHERE 절 에서는 AND 연산으로 이어진 경우에 다음 인덱싱을 사용하는게 가능하다. 그래야 기존 인덱스 범위를 사용하는게 가능하기 때문에

  - OR 절을 사용하는 경우에는 index merge 가 일어난다. (싱글 테이블의 경우에만)

- 인덱스를 사용하는데 조회하는 범위가 일정 범위를 넘어간다면 인덱싱을 사용하지 않는다. 굳이 그럴 필요가 있나 싶어서

- 읻덱스 칼럼의 값을 가공하는 경우에는 인덱스를 적절하게 사용하지 못한다.

- 인덱스를 이용하는 경우 상수의 데이터 타입과 같아야한다. 그렇지 않으면 타입을 맞추기 위해서 타입 변환한 후 비교작업을 처리하는데
이러면 인덱스 값 비교를 바로 할 수 없기 때문에.

- index merge 는 인덱스 range query 를 합치는건데 싱글 테이블의 경우에만 가능하다.

  - 만약에 복잡한 where 절을 써서 optimal 한 결정을 데이터베이스가 하고 있지 않다면 다음과 같이 식을 변형해봐라.
  
  ```sql
  (x AND y) OR z => (X OR Z) AND (Y OR Z)
  (X OR Y) AND Z => (X AND Z) OR (Y AND Z)      
  ``` 
  
  - 실행 계획을 보는 EXPLAIN 절에서 Index Merge 를 하는 경우에는 `index_merge` 라는 걸 볼 수 있다.  

 
***

## Multiple-Column Indexes

MySQL 에서는 최대 16개까지 칼럼을 합쳐서 인덱스로 사용하는게 가능하다.

실제 쿼리에서는 첫번째 칼럼부터 사용한다.

Composite 인덱스 대신에 여러 칼럼을 해싱한 인덱스를 사용하는 것도 가능하다. 이 경우에는 여러 열을 사용하는것보다 빠를 수 있다. 

예시를 보면 다음과 같다. 

#### test 테이블 
```sql
CREATE TABLE test (
    id         INT NOT NULL,
    last_name  CHAR(30) NOT NULL,
    first_name CHAR(30) NOT NULL,
    PRIMARY KEY (id),
    INDEX name (last_name,first_name)
);
```

```sql
SELECT * FROM test WHERE last_name='Jones';

SELECT * FROM test
  WHERE last_name='Jones' AND first_name='John';

SELECT * FROM test
  WHERE last_name='Jones'
  AND (first_name='John' OR first_name='Jon');

SELECT * FROM test
  WHERE last_name='Jones'
  AND first_name >='M' AND first_name < 'N';
```

- lastname 만으로도 인덱싱을 할 수 있다. 가장 왼쪽 값이므로.

다음과 같은 조회는 인덱스를 사용할 수 없다.

```sql
SELECT * FROM test WHERE first_name='John';

SELECT * FROM test
  WHERE last_name='Jones' OR first_name='John';
``` 

***

## Verifying Index Usage

EXPLAIN 절을 사용하면 정말로 인덱스를 사용하는지 볼 수 있다.

***

## Index Merge Algorithm 

#### Index Merge Intersection Access Algorithm

Where 절에서 AND 연산으로 연결될 때 사용하는 알고리즘이다. 

쿼리에 있는 모든 칼럼을 인덱스로 커버가 가능하면 인덱스로만 가져오고 전체 행을 가져오지 않는다. 

만약에 쿼리에 있는 모든 칼럼이 인덱스로 커버가 가능하지 않다면 그 인덱스에 해당하는 범위만 가져온다. 

Primary Key 는 여기서 검색을 위한 조건보다는 필터를 위한 조건으로 사용된다.
 
예시는 다음과 같다. 

```sql
SELECT * FROM innodb_table
  WHERE primary_key < 10 AND key_col1 = 20;

SELECT * FROM tbl_name
  WHERE key1_part1 = 1 AND key1_part2 = 2 AND key2 = 2;
```
#### Index Merge Union Access Algorithm

Where 절에서 OR 연산으로 연결될 때 사용되는 알고리즘이다. 

예시는 다음과 같다. 
```sql
SELECT * FROM t1
  WHERE key1 = 1 OR key2 = 2 OR key3 = 3;

SELECT * FROM innodb_table
  WHERE (key1 = 1 AND key2 = 2)
     OR (key3 = 'foo' AND key4 = 'bar') AND key5 = 5;
```

#### Index Merge Sort-Union Access Algorithm

Index Merge Union 알고리즘과의 차이점은 Sort-Union 알고리즘은 Union 알고리즘 보다 먼저 첫번째로 전체 행의 id 를 뽑아내서 정렬을 한다. 

예시는 다음과 같다. 

```sql
SELECT * FROM tbl_name
  WHERE key_col1 < 10 OR key_col2 < 20;

SELECT * FROM tbl_name
  WHERE (key_col1 > 10 OR key_col2 = 20) AND nonkey_col = 30;
```

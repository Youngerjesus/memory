## Optimizing Sub queries

SubQuery 에서 `IN, =, ANY, or EXISTS` 를 사용하는 경우에 Optimizer 가 선택할 수 있는 선택은 다음과 같다. 

- Semijoin

- Materialization

- `EXISTS` strategy

SubQuery 에서 `NOT IN, <>, ALL, NOT EXISTS` 를 사용하는 경우에 Optimizer 가 선택할 수 있는 선택은 다음과 같다.

- Materialization

- `EXISTS` strategy 


### 8.2.2.1 Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations

세미조인은 조인을 했을때 매칭되는 행의 수가 상관 없는 경우에 딱 매칭되는 행이 한개만 필요하면 되는 경우에 쓰면 좋다.
기본적으로 OneToMany 에서 INNER JOIN 을 한다고 헀을 때 매칭되는 행이 여러개가 생길 수 있는데 이게 필요하지 않은 경우 말이다. 

예를 들면 `Country` 와 `City` 가 있다고 할 때 1000만 명 이상의 인구가 있는 도시를 포함하고 있는 나라들을 나타내라!
`County` 와 `City` 를 조인하게 되는데 나라 입장에서 모든 도시들을 조인할 필요가 없다. 1000만 명 이상의 사람을 포함하고 있는
빅 시티를 찾고나면 그 이후로 실행을 안해도 된다. 

그리고 세미 조인의 실행과정은 `backwards` 로도 가능한데 `City -> Country` 로 조인을 하는 걸 말한다. 
이 방법으로 실행을 하면 `City` 를 풀스캔하면서 빅 시티를 발견하면 `Country` 로 조인해서 결과를 만든다.

MySQL 8.0.17 을 기준으로 다음과 같은 서브쿼리는 `antijoin` 을 실행한다.

- `NOT IN (SELECT ... FROM ... )`

- `NOT EXISTS (SELECT ... FROM ...)`

- `IN (SELECT ... FROM ...) IS NOT TRUE` 

- `EXISTS (SELECT ... FROM ...) IS NOT TRUE`

- `IN (SELECRT ... FROM ...) IS FALSE`

- `EXISTS (SELECT ... FROM ...) IS FALSE`

##### Antijoin 예

```sql
SELECT class_num, class_name
    FROM class
    WHERE class_num NOT IN
        (SELECT class_num FROM roster);
```

- 여기서 안티조인은 클래스를 쭉 훑으면서 `class_num` 과 같은 값이 `roster` 에 있는지 검사한다. `roster` 가 `class_num` 이 
  인덱스로 걸려있다면 쉽게 찾고 `class` 에 있는 그 행을 버린다. 그런식으로 결과를 만든다. 
  
   
### 8.2.2.2 Optimizing Subqueries with Materialization

Materialization 방법은 메모리에 임시 테이블을 만듬을 통해서 쿼리 실행 속도를 올린다. 이 테이블은
해시 기반의 테이블로 빠르게 참조가 가능하다. 

- 어떤 값을 인덱스로 만드는지 궁금하네. 

#### Noncorrelated and Correlated Subqueries

https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/AnalyzingData/Queries/Subqueries/NoncorrelatedAndCorrelatedSubqueries.htm

##### Noncorrelated subquery

- 바깥의 outer statement 와 독립적으로 inner statement 에서 실행되는 걸 말한다. 

- 실행 순서는 inner statement 를 먼저 실행하고 그 결과를 outer statement 에 던져준다. 

##### Correlated subquery 

- outer statement 에서 inner statement 에서 사용할 것들을 먼저 뽑고나서 inner statement 에서 적용하는 것   


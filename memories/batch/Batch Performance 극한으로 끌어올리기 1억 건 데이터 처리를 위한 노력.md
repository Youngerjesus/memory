# Batch Performance 극한으로 끌어올리기: 1억 건 데이터 처리를 위한 노력

- 어떻게 batch performance 를 개선할 수 있을까? 

- 실제 개선을 할 땐 어느 것이 bottler neck 인지알고 있어야한다. 
  - Reader - Process - Writer 로 처리되고 있다면 Reader 가 큰 지, Processor 가 큰 지, Writer 가 큰 지 알고 있어야함.
  - 그리고 해당 부분을 개선하면 된다. 
  - 일반적으로는 Reader 의 비중이 크다. 

## Reader 개선

### Pagination Reader 를 사용할 떄 주의. 

```sql 
SELECT * FROM orders
WHERE category = 'BOOK'
LIMIT 3600, 100 
```

- pagination 과 chunk process 는 궁합이 잘맞다. 근데 limit 을 이용한 pagination 은 주의하자. 
- limit 를 이용한 offset 처리는 느리다. offset 이 커질수록 느리다. (여기선 3600 값보다 커질수록 쿼리 실행속도는 더 느려진다.)
  - 왜냐하면 offset 만큼의 개수를 다 읽어야하니까. 

- 그래서 이를 개선하려면 zero offset 이 되도록 읽어야 하는 곳을 명확히 해줘야한다.  

```sql
# page-1번 쿼리 
SELECT * FROM orders where category = 'BOOK' and id > 0 limit 0, 100;

# page-2번 쿼리
SELECT * FROM orders where category = 'BOOK' and id > 5235 limit 0, 100;

# page-3번 쿼리
SELECT * FROM orders where category = 'BOOK' and id > 967678332 limit 0, 100;
```


#### limit 에 대해서 더 알아보자. 

limit 은 반환될 개수가 충족되면 쿼리는 성공적으로 종료된다.

```sql
SELECT * FROM employees
WHERE emp_no = BETWEEN 10001 AND 100000
ORDER BY first_name 
LIMIT 0, 5; 
```

1) where 절이 먼저 실행되서 전체 집합을 가져오고, 

2) order_by 로 정렬한다. 근데 여기서 limit 의 개수인 5개가 충족되면 쿼리는 종료된다. 


GROUP_BY 와 같이 쓰인 LIMIT 는 LIMIT 이 모두 끝나야 한다. 
```sql
SELECT * FROM employees 
GROUP BY first_name 
LIMIT 0, 10
```

## Aggregation 비용을 줄이자. 

- 원칙은 DB 에서 모든 Aggregation 을 하지말고 group_by 나 sum 같은 연산은 어플리케이션 단계에서 하자. (쿼리를 단순하게 만들기 위해서)
  - 모든 리소스는 놀고 있으면 안되고 사용중이어야 효율적이다. 어플리케이션에서도 가능하고, DB 에서도 가능하다면 한쪽에 너무 많은 부담을줘서 놀고 있게 하지 말고, 부담을 좀 나눠서 같이 처리해나가면 더 효율적일듯
  - 이런 원리로 groupby 와 집계 메소드를 DB 에서 어플리케이션으로 옮기지 않았을까?

- 여기서는 데이터를 chunk 로 쪼게서 Redis 에서 집계하도록 했다.
  - 메모리에서 집계 할 수 있어서 연산도 지원해주고.
  - 대신 I/O 가 많이 일어날 수 있다. 
    - I/O 가 몇번 일어나는지도 계산해버리네. (1ms 정도 걸린다고 생각하고 I/O 가 일어나는  개수를 곱해서 총 걸리는 시간을 예측하는 것.)
    - Redis 에서는 Pipeline 을 통해서 command 를 묶어서 보낸다.

## Writer 개선 

- Batch Insert
- 명시적 쿼리 (= 영속성 컨택스트 쓰지 않기.)

## Spring Cloud Data Flow 에서 Batch 실행 

- 그냥 Spring Batch 만 쓰면 로그 정보가 빈약해서.
- Batch 의 상태 파악이 어려워서

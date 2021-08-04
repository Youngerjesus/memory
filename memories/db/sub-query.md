## SubQuery

서브쿼리는 SQL 내부에서 작성되는 일시적인 테이블이다. 이를 영속화해서 보여주는 것이 뷰이지만
뷰는 데이터를 저장하지는 않는다. 

기본적으로 서브쿼리는 쿼리안에 또다른 쿼리가 있는 걸 말한다. 

서브쿼리는 쓰이는 위치에 따라 3가지 종류로 나뉘는게 가능하다.

- SELECT 절과 ORDER BY 에 쓰이는 scalar subQuery 가 있다

- FROM 절에 쓰이는 INLINE_VIEW

  - 따로 Object 를 만들지 않고 일회성으로 만드는 VIEW 이다.

- WHERE 절 HAVING 절에 쓰이는 중첩 서브쿼리가 있다.

  - WHERE 절에 서브쿼리를 쓸 때 여러번의 테이블 스캔 OR 여러번의 인덱스 스캔을 할 수 있다. 
    이 경우 여러번의 스캔을 하는게 비용이 크다면 다른 방식을 쓰도록 해서 개선하는 걸 추천한다. 
    한번의 스캔으로 끝낼 수 있는지
    
테이블과 서브쿼리는 기능적인 관점에서는 전혀 차이가없다. 데이터베이스도 딱히 자신이 다루는 오브젝트가
테이블인지 뷰인지 구별하지 않는다. 하지만 비기능적인 요소에서 차이가 나는데 이를 알아보고 어떤 점을 신경써야 하는지 알아보자.

#### 서브쿼리가 가지는 문제점

서브쿼리가 지니는 문제점은 테이블을 쓰지않고 일시적으로 뷰를 만든다는 점에서 기인한다. 

실제적인 데이터를 가지고 있지 않기 때문에 서브쿼리에 접근할 때마다 SELECT 구문을 통해 데이터를 만들어야 하는 연산 비용이 들어간다.

또 다른 비용으로는 연산 결과를 이용해 데이터를 만들었으면 이를 임시적으로 저장해야한다. 메모리 여유공간이 있다면
메모리에 저장하면 되지만 만약 결과가 크다면 디스크에 저장해놔야 하는 경우도 생긴다 그러면 I/O 비용이 크게 된다. 

마지막으로 테이블 같은 경우는 명시적인 제약이나 인덱스 같은 정보가 충분히 있지만 서브쿼리 같은 경우는 임시적으로 만드는거니까
인덱스 정보나 제약조건 정보를 얻기힘드므로 최적화를 하기가 힘들다.

#### 서브쿼리의 폐헤 

서브쿼리를 통해 테이블 엑세스를 많이 해야하는 경우가 있다. 

#### 서브쿼리는 그러면 언제 사용할까? 

조인 해야하는 레코드 수를 줄여줄 때 

#### SubQuery Example 1

##### min() max() 에 subQuery 를 쓰는 경우 - 여러번의 인덱스 스캔이 나감  

```sql
select * from sample.employees A
	where A.officeCode = (select max(officeCode) from sample.employees)
		OR A.officeCode = (select min(officeCode) from sample.employees);
```

##### 한번의 테이블 스캔과 정렬을 이용한 경우 

```sql
select	B.lastName,
		B.firstName,
		B.officeCode,
        B.minOfficeCode,
        B.maxOfficeCode
	from (
		select  A.lastName,
                A.firstName,     
                A.officeCode, 
                A.jobTitle,
                row_number() over(order by officeCode) minOfficeCode, 
                row_number() over(order by officeCode desc) maxOfficeCode
		from sample.employees A
	) B
	where B.minOfficeCode = 1 OR B.maxOfficeCode = 1;
```

 






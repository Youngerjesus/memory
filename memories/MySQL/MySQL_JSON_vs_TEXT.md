# MySQL JSON vs TEXT

https://medium.com/daangn/json-vs-text-c2c1448b8b1f

## Intro

- MySQL 이 JSON 타입을 지원하기 시작했는데 JSON 타입을 쓰는게 좋을까? TEXT 타입을 쓰는게 좋을까? 를 비교하는 것.

## 성능 테스트 비교

TEXT 와 JSON 칼럼을 다음과 같은 SQL 문에 대해서 성능을 비교함.

- `SELECT COUNT(id)`

- `SELECT COUNT(data)`

    - JSON Parsing 작업이 필요하진 않음.

- `SELECT COUNT(data->>'$.field'`

    - JSON Parsing 작업이 필요함. 파싱은 일부만 하지는 않는다. 전체를 다한다.

- `SELECT id`

- `SELECT data`
    - 이 테스트는 클라이언트로 데이터를 가지고 오는 테스트.
    - 여기서는 TEXT 칼럼이 JSON 칼럼보다 성능이 3배정도 좋았음. (데이터 16,000 건 수백 KB 기준.)
    - TEXT 는 문자열 변환만 하면 되지만 JSON 은 파싱 후 문자열로 해석하는 Serialization 과정을 거쳐야함. (파싱만 하면 내부적으로 Binary JSON Dom 으로 변환됨.)

## 기능 비교

JSON 타입을 쓰면 다음과 같은 기능을 제공함

- 특정 필드의 값 조회 및 변경 가능 (TEXT 는 통쨰로 바꿔야함.)
- 특정 필드 업데이트시 in-place 업데이트 가능 (변경되는 필드가 고정길이라면 in-place 업데이트를 통해서 디스크 블록 통째로 교체하지 않아도됨)
- 특정 필드에 대해서 인덱스 생성 가능.

## 언제 JSON 타입을 쓰는가

- 특정 필드의 값만 접근, 변경 가능해야하는 경우
- 특정 필드 (고정길이) 업데이트를 자주 하는 경우
- 특정 필드에 대해서 인덱스를 생성이 필요한 경우

## 그리도 또 하나 더

온라인 트랜잭션 (OLTP) 의 경우 크기가 큰 데이터를 조회하면 대역폭 문제, CPU 문제, 느린 응답 속도, 메모리 모두 문제가 될 수 있다.

OLTP 에선 데이터를 최대한 컴팩트하게 유지하는게 좋다.

만약 크기가 큰 데이터를 이용해야한다면 다른 DB 를 이용하거나, DBMS 이외에 다른 저장소를 이용하는 것도 고려해볼만함.





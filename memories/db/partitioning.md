## Partitioning 

파티셔닝은 데이터베이스의 Write 가 많아져서 성능이 안나올때 Write 를 줄이는 목적으로 사용하는 걸 말한다. 

특정 Id 나 특정 연산을 기반으로 데이터베이스를 라우팅 해주는 것. 

파티셔닝 할 때 중요한 건 관리 포인트를 어떻게 할 것인가? 

- 특정 데이터베이스가 장애가나서 복구할 때 동안 어떻게 처리할 것인가?  
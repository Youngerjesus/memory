# Cache Aside 

https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside

*** 

## 이 패턴을 쓰는 이유 

- 퍼포먼스를 내기 위해서

## 주의사항 

- 캐시와 데이터 스토어의 일관성은 안맞을 수 있다. 이걸 어떻게 가능한 빠르게 갱신해줄 것인지에 대한 전략. 

- cache data 의 만료시간 

- evicting data 
  - 데이터를 지우는 정책 

- startup 시간에 캐시를 채우는 것. 

- consistency 

- local caching
  - expire 와 refresh 정책이 필요하다.



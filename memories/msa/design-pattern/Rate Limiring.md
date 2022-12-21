# Rate Limiting 

https://learn.microsoft.com/en-us/azure/architecture/patterns/rate-limiting-pattern

***

## 이 패턴은 뭔가? 

- throttling pattern 과 유사한 점이 많음. 
  - 대신에 throttling 에서 에러를 만나기 싫거나, 최소화 하고 싶은 것. 
    - 배치 프로세싱에 적합하다. 
    - 처리가 실패되면 또 엄청난 양의 로그를 남겨야함.
  - throttling 은 request 를 reject 하고 retry 를 하도록 강요. 이를 통해서 resending 데이터가 많이 들어올 것.
    - 실제 처리되야하는 데이터는 1만건인데, retry 를 통해서 더 많은 데이터를 보내는 걸 강요받을 수 있다.

## 이 패턴을 적용할려면? 

- durable 한 messaging system 이 있어야한다. 
  - 여기서 message 들을 몰아넣고, jobProcessor 들을 통해서 속도를 제어하면서 메시지를 꺼내서 throttled service 에 던져주는거지.
    - 그냥 job processor 를 없애고 꺼내가는 쪽에서 속도를 제어하면 되는거 아닌가? 
  




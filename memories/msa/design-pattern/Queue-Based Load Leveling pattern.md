# Queue-Based Load Leveling pattern

https://learn.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling

***

## 이 패턴을 쓰게 되는 경우 

- 간헐적인 무거운 부하로 인해서 service 가 실패하거나, task 가 타임아웃 날 수 있는데 부하를 완하 시키기 위해서.
  - 가용성과 응답성을 보장하면서
- 보편적으로 적용할 수 있음. 
- minimal 한 latency 가 필요하지 않다면. 

## 어떻게 이 패턴을 적용하는지? 

- task 와 service 사이에 queue 를 두는 것. 
  - queue 는 buffer 로 쓰임. 
  - 비동기 처리 적용

## 고려사항 

- 처리하는 서비스의 수와 큐의 숫자.
- 부하를 다음 시스템의 stage 로 넘어가는게 문제가 있진 않은지.  
- 응답이 필요한 구조는 힘들다. one-way 통신이라서. 
  - 응답이 필요하다면 `Asynchronous Messaging Primer` 를 쓰라고 하는듯.
  - 여기서 말하는 응답은 내가 보낸 메시지가 잘 전달되었는지를 말함.

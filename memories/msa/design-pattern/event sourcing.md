# Event Sourcing 

https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing

***

## 이 패턴은 뭔가? 

- 현재 상태의 데이터를 넣는게 아니라, 데이터가 변하는 상태들을 기록하는 것. 
  - append-only store 이어야함.
  - record 처럼 행동함. 
- 데이터를 CRUD 로 관리하는 형태와는 반대임.

## 이 패턴은 왜쓰나

- 데이터 모델에 대한 동기화를 피할 수 있으면서 성능과 확장성 응답성을 주기 위해서. 
  - CRUD 의 update 보다 append-only 가 더 빠르다. 그리고 많은 유저가 쓰는 경우에 동시성 제어도 필요하다.
  - 쓰기가 더 필요하고, 일관성이 중요할 떄 나름의 이유가 될 수 있을듯. 
- compensating actions 을 할 수 있는 full audit trail and history 를 얻을 수 있음.
- 데리터를 캡처하고 싶을 떄 

## 이 패턴은 어떻게 쓰는건가? 

- 각각의 데이터에 대한 operation 들을 event 들로 기록해놓는다.
  - 이벤트는 데이터들에 대한 변화다. 
  - 이벤트의 주 사용은 엔터티에 대한 구체화된 뷰 (materialized view) 를 유지시켜놓는것.
    - 예를 들면 주문 UI 에서 보여줄 모든 주문 정보들.
      - 주문 속에는 item 이 들어오고 나오고 할텐데 이런 이벤트들이 다 모여서 주문을 만드는 거임/
- 어플리케이션은 이벤트들의 시리즈를 보내는 것.
  - 이벤트를 내보내면 이 이벤트를 subscribe 하는 쪽에서 받아서 operation 을 한다.


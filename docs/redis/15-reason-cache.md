## 15 Reasons to Use Redis as an Application Cache

***

#### Summary 

캐시를 쓰는 이유 자체는 자주 사용하는 데이터를 캐시해서 어플리케이션의 응답 시간을 줄이기 위해서 사용한다.

- 매 요청마다 외부 데이터 소스에 접근하는게 아니라. 케시에 있는 데이터를 전달하는 방식으로


#### Cache 에 담을 데이터는 뭘까? 

- 설정 정보들(Configuration settings) 

  - 설정 정보를 disk storage 에 있는 텍스트 파일에서 데이터를 읽어오는 경우보다 캐시에서 읽어오는게 더 빠르다. 

-  현지화 및 국제화 데이터(Localization and internationalization data)

    - 유저 친화적인 어플리케이션은 국제화와 관련된 정보를 제공해주는데 이런 국제화 정보는 외부 저장소에 저장되는 경우가 많다.
    (관심사를 나누기 위해서 그러는 듯) 이런 정보를 캐싱해서 사용하면 응답 시간을 줄일 수 있다.

-  템플릿과 부분적인 렌더링 응답(Templates and partially rendered responses)

    -  

  

  

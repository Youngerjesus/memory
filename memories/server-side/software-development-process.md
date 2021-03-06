# 소프트웨어 개발 프로세스

***

## 애자일 개발 방법론

### 소프트웨어 개발의 정의 

요구사항을 분석해서 디자인하고 이 디자인을 각각의 개발자들이 나눠서 개발을 진행한다. 

개발된 컴포넌트는 테스트되고 마지막에는 합쳐져서 하나의 유기적인 시스템을 만든다. 

이러한 개발 과정을 '개발 프로세스' 라고 하고 여기에 조직 구조와 도구셋을 합쳐서 '개발 방법론' 이라고 한다. 

즉 개발 방법론은 개발 프로세스를 어떠한 개발 조직과 도구들을 가지고 이뤄내는지를 말한다. 

소프트웨어 개발은 Waterfall 방식의 개발 모델에서부터 근래에는 애자일 방식까지 계속해서 진화해왔다. 

이 진화의 철학은 한가지다. `얼마나 효율적으로 개발 프로세스를 진행할 수 있는지.`

소프트웨어 개발은 사람이 하는 일이다. 그러므로 계속해서 요구사항은 변한다.

이렇게 소프트웨어 개발은 이런 요구사항이 변한다는걸 전제하에 변화를 수용하고 이에 맞게 대처하는 형태로 변해왔다. 

이런 방법들은 `실용주의 방법론(Practical Methodology)` 라고 한다. 

싱용주의 방법론의 특징은 다음과 같다. 

- 변화를 수용한다. 

  - 요구사항은 언제나 변할 수 있고 이에 맞춰서 개발 범위를 조정하면서 프로젝트를 진행한다. 

- 개발 과정을 짧은 조각으로 나누어서 반복적으로 진행한다.

  - 반복적 (Iterative) 개발 방법론 이라고도 하는데 요구사항 분석 - 설계 - 개발 - 테스트 이 라이프 사이클을 짧게 가져가는 걸 말한다.
  그래서 한 주기가 끝날 때마다 프로세스를 성숙시킨다.   
  
- 소프트웨어 개발에 있어서 결함은 존재 할 수 있다를 전제로 가져간다. 

  - 결함이 존재할 수 있고 이런 결함은 테스트 코드를 통해서 찾아낼 수 있도록 한다. 
  
- 문서 작업을 최소한으로 한다. 

  - 서로 커뮤니케이션을 원활하게 할 수 있는 문서를 사용하도록 하고 될 수 있으면 산출물 형태의 문서는 생략한다. (아무래도 자주 바뀔 수 있다는 특징이 있으니까 그런듯.)
  대신 온라인 문서인 Wiki 를 통해서 주로 작성한다. 
 
- 협업과 커뮤니케이션에 많은 비중을 할당한다. 

  - 애자일 방법론 중의 하나인 Extreme Programming 에서는 고객을 프로젝트 공간에 같이 둔다. 그 이유는 요구사항을 잘못 이해하면 생기는 비용이 너무나 크기 떄문인데 소프트웨어 개발 프로세스의 과정을 다시 해야하기 때문이다.
  이와 같은 문제를 만나지 않기 위해서는 요구사항을 정확하게 이해하는게 중요하다. 
  
- 자동화 도구를 사용한다. 

  - 빌드나 테스트를 통합된 환경에서 자동화 하도록한다. (소프트웨어 개발의 시간을 줄이는게 목표인듯.)
  
이러한 실용주의 개발 방법론은 프로젝트 기간이나 일정을 변경해버릴 수 있기 떄문에 통제하기가 어렵다. 

이를 위해서 `변형된 스크럼 방법론` 과 함께 `JIRA 를 이용한 일반적인 스크럼 개발 방법론` 을 소개한다. 

### 애자일 개발 방법론이 뭔데?

실용주의 개발 방법론이 애자일을 통해서 이뤄지는데 이 방법의 가장 큰 특징은 협업(Collaboration) 을 통해서 이뤄진다는 점이다. 

이 방법은 협업과 테스트(Task) 를 관리하는 기법을 통해서 이뤄진다. 

애자일 방법론은 여러가지 구현체들이 있는데 이 중에는 다음과 같은 것들이 있다.

- Extreme Programming

- AUP (Agile Unified Process)

- 스크럼

- lean 등 

이 중에서는 스크럼이 가장 인기있고 많이 사용된다. 

스크럼은 기본적으로 반복과 점진적(Iterative and Incremental) 개발 방법에 기초한다. 

스크럼은 몇개의 이터레이션으로 구성되고 각 이터레이션을 스프린트라고 한다. 

각 스프린트는 1 ~ 4 주의 기간을 가지며 이 기간은 변경될 수 없음을 전제로 가져간다. 

스크럼에는 고객의 요구조건으로 부터 작성된 제품 백로그 (Product Backlog) 라는 걸 가진다. 

쉽게 이야기하면 대략적인 할 일 목록이다. 소프트웨억 객발 단계 중 구현 단계에서 스크럼을 적용할 경우에는 요구사항 명세서나 기술요구사항 정의서가 제품 백로그에 해당될 것이다. 

이 제품 백로그 항목을 팀 미팅을 통해서 이번 스프린트에서 가져갈 사항으로 우선순위 기반으로 정한다.

그 다음 이 스프린트를 위해서 구체적으로 해야할 테스크들을 정하고 이 목록 리스트를 스프린트 백로그 (Spring Backlog) 라고 한다. 

스프린트 백로그의 테스크들은 스케쥴이 지정되고 담당자에게 할당된다. 

담당자는 이 스프린트 백로그를 가지고 정해진 기간 동안 스프린트를 구현한다. 

이렇게 정해졌으면 매일 아침동안에 10분 ~ 20분 동안 일일 스크럼 미팅(Daily Scrum) 이라는 회의 시간을 가지고 오늘 한 일에 대해서 간략하게 보고한다. 

이를 통해서 현재 스프린트에서 어느정도 진행상황인지를 파악한다. 

스프린트가 끝나면 해당 소프트웨어를 릴리즈한다. 릴리즈가 끝난 후에는 간단한 리뷰 미팅을 가지는데 구현한 내용을 간략하게 데모 형태로 소개하고 피드백을 받아서 제품 백로그에 업데이트 하고 다음 스프린트를 구현한다. 

***

## 엔터프라이즈 개발을 위한 스크럼 기반의 개발 방법론 

스크럼 방법론을 통해서 개발 단계별 진행상황을 정확하게 아는게 가능하다.
 
### 제품 백로그 준비 

제품 백로그는 실제로 구현되어야 하는 목록으로 고객의 요구사항을 기반으로 도출된다. 

이러한 제품 백로그는 다소 모호하게 작성되면 안되고 종료조건이 명확해야한다. 

제품 백로그에 있어야 하는 사항으로는 다음과 같다.

- 번호

  - 기능에 대한 ID
  
- 항목 제목 

  - 구현 요건 (기능/비기능)
  
- 설명 

  - 요건에 대한 간략한 설명
  
- 예측 소요 시간 

  - 얼마나 많은 리소스가 소요되는가에 대한 예측치를 기록한다. 
  
- 요건 설명 맄크

  - 부족한 설명은 문서에 대한 링크로 설명
  
- 우선순위 

- 상태

- 항목에 대한 비즈니스 가치  
 




## Spring MVC

#### MVC 아키텍처

뷰에서 렌더링할 데이터를 데이터베이스에서 가져오는 역할을 하는 Model 화면을 담당하는 View 요청을 처리하고 제어하는 Controller 로 계층을 나눈
패턴을 말한다. 

MVC 패턴을 사용하지 않으면 관심사를 나누기 힘드니까 화면을 담당하는 코드, 데이터베이스 엑세스하는 코드, 요청을
처리하는 코드가 모두 섞이게 되므로 이렇게 하면 안되니까 MVC 패턴이 나왔다.  

#### 프론트 컨트롤러 패턴 

사용자의 요청을 가장 처음에 만나는 프론트 컨트롤러를 두고 공통적인 처리를 해준 다음에 
세부적인 요청을 처리할 컨트롤러를 찾아서 처리하는 패턴이다. Locale 정보를 보고 처리하고
Multipart 요청인 경우 Multipart Resolver 가 처리하도록 한다거나  

스프링 MVC 는 프론트 컨트롤러 패턴 + MVC 패턴으로 이뤄져있다.

#### 필터란?

필터는 클라이언트 요청이 서블릿으로 가기전에 먼저 처리할 수 있도록 WAS 에서 지원하는 기능이다. 

가장 대표적인 필터로는 인코딩 필터가 있다. 

서블릿 필터는 서블릿 스펙을 기반으로 만들어지므로 서블릿 컨테이너에서 생성되고 실행된다. 

스프링 빈을 서블릿 필터 스펙에 맞게 구현하더라도 스프링에서 정의된 빈을 주입해서 사용하는게 불가능하다.

필터를 통해서 서블릿에게 들어가고 필터를 통해서 나가게된다. 

#### 요청을 처리하는 과정 

DispatcherServlet 이 Servlet 과 연결되어 있다. (스프링 부트는 내장 톰캣 안에 DispatcherServlet 을 등록한다.)

스프링 시큐리티를 사용하고 있다면 서블릿 컨테이너에 있는 서블릿 필터 중 하나인 DelegatingFilterProxy 가 
스프링 컨테이너에 있는 FilterChainProxy 로 빈 이름이 springSecurityFilterChain 이라는 걸 ApplicationContext 에게 찾아서 위임해준다. 
이후에 FilterChainProxy 가 가지고 있는 Filter 들을 연쇄적으로 처리한 다음에 DispatcherServlet 에게 넘긴다. 

DispatcherServlet 이 먼저 처음 사용자의 요청을 받고나서 공통적인 처리를 한 후
URL 정보나 HTTP METHOD 정보등을 바탕으로 어떤 핸들러가 처리해야할 지를 결정해야한다. 
핸들러를 선택할 수 있는 핸들러 매핑 전략에 따라 핸들러를 고르도록 하고 실제 핸들러가 가진 메소드 이름을
DispatcherServlet 이 알 지 못하니까 Adapter 패턴을 통해서 실제 컨트롤러를 호출한다.
컨트롤러는 요청을 처리하고 응답이 있다면 ModelAndView 를 리턴하고 뷰 리졸버는 이 정보를 바탕으로
사용자에게 응답해줄 응답 객체를 만들어서 전달해준다. 

#### 핸들러 매핑과 핸들러 어댑터 

핸들러 매핑은 실제 요청을 처리할 컨트롤러를 고르는 작업이고 핸들러 어댑터는 핸들러 매핑을 통해 고른 컨트롤러의 
메소드를 모르니까 Adapter 패턴을 통해 호출하는 방식을 말한다.

핸들러 매핑의 종류로는 BeanNameUrlHandlerMapping 과 RequestMappingHandlerMapping 이 있다.

#### 핸들러 인터셉터

핸들러 인터셉터는 핸들러를 실행하기 전과 후에 요청과 응답을 가공하는 필터를 말한다.

- preHandle()

  - 핸들러를 실행하기 전에 호출된다.
  
- postHandle()

  - 핸들러를 실행하고 뷰를 렌더링하기 전에 호출된다. 
  
  - 뷰에 전달할 공통적인 모델 정보를 담는데 사용할 수도 있다. 
  
  - 비동기 요청 처리에는 실행되지 않는다.
  
- afterCompletion()

  - 요청 처리가 완전히 끝난 뒤에 호출된다. preHandle() 에서 true 를 리턴한 경우에만 호출된다.
  

#### DispatcherServlet 의 구성요소는?

- MultipartResolver

  - 파일 업로드 요청시 필요한 인터페이스 

- LocaleResolver

  - 클라이언트의 위치 정보를 파악하고 이를 처리해주는 인터페이스 

- ThemeResolver

  - 애플리케이션 설정 테마를 파악하고 변경하는 인터페이스 

- HandlerMapping

  - 애플리케이션 핸들러를 찾는 인터페이스

- HandlerAdapter

  - 핸들러 매핑이 찾아낸 핸들러를 실행하는 인터페이스 

- HandlerExceptionResolvers

  - 요청중에 발생한 에러를 처리하는 인터페이스 

- RequestToViewNameTranslator

  - 핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우 요청을 기반으로 뷰 이름을 판단하는 인터페이스 

- ViewResolvers
  
  - 뷰 이름에 해당하는 뷰를 찾아내는 인터페이스  
  
- FlashMapManager

  - FlashMap 인스턴스를 가져오고 저장하는 인터페이스 



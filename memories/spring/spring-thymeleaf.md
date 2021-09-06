# Spring Thymeleaf 

큰 틀에서 부터 설멸하면서 -> 디테일하게 들어가는 식으로 

그 다음 마무리는 추가로 더 깊게 보고 싶다

## Thymeleaf 가 나온 이유

## Thyemeleaf 의 특징

## Thymeleaf 와 JSP 와의 차이점

## Spring Thymeleaf 의 Data Flow

### DispatcherServlet

1. doDispatch()
2. HandlerAdapter.handle() : ModelAndView
3. processDispatchResult (processedRequest, response, mappedHandler, mv, dispatchException)
4. render(mv, request, response)
5. resolveViewName

여러가지 ViewResolver 들을 가지고 있다. 

- ContentNegotiationViewResolver
- BeanNameViewResolver
- ThymeleafViewResolver
- ViewResolverComposite
- InternalResourceViewResolver

## 핵심 컴포넌트들

### FrameworkServlet

- doService() 메소드
- publishRequestHandledEvent
- DispatcherServlet 의 인터페이스 구나. 각각의 역할에 대해서도 알아야 할 듯.

### DispatcherServlet

- dispatch 뜻이 뭔데? 신속히 일을 발생하다. 이런뜻

- doDispatch() 부터 시작한다. 

  - 핸들러 매핑 전략에 따라서 적합한 핸들러를 찾고 핸들러 어댑터를 통해 요청을 처리할 핸들러를 호출한 후 결과로 ModelAndView 를 받는다. 
  
    - ModelAndView 는 뭔데? ModelAndView 는 MVC Framework 의 Model 과 View 둘의 데이터를 가지고 있는 객체다. 
    
    - ModelAndView 에 있는 Model 은 Map 형식으로 이뤄져있어서 여러개의 객체 데이터를 넣을 수 있고 View 는 이 값을 통해서 ViewResolver 가 뷰 파일을 찾고 렌더링 할 수 있다.  
   
- processDispatchResult()

  - 요청을 처리할 핸들러를 찾고 핸들러를 실행한 후 나온 결과를 처리해주는 메소드다. 
  
- render() 

  - ModelAndView 가 있다면 주어진 이 데이터를 바탕으로 ViewResolver 가 적합한 View 를 찾고 렌더링한다. 

- resolveViewName()

  - 주어진 뷰 이름을 바탕으로 ViewResolver 를 통해 View Object 를 찾는 메소드다 
  
  - 기본적인 View Object 를 찾는 전략은 DispatcherServlet 이 가지고 있는 여러개의 ViewResolver 를 통해서 묻는 식으로 진행된다. 
  이 전략은 커스텀하겍 구현하는게 가능하다.  
  
  - 찾은 View 를 통해서 렌더링 하도록 한다.  

- View.render() 

### TemplateEngine

- Template 실행과 관련한 Main Class 이다.
- ThymeleafView 에 render() 메소드 내부에서 TemplateEngine.process() 를 호출한다.

- TemplateResolver 가 가져온 Template 을 처리해주는 역할을 한다.
    - 어떤 처리를 말하지?
        - Template 은 Dialect  설정에 따라서 처리해준다.
    - Template 의 정의는 문서의 시작점 역할을 해주는 파일이다.

- TemplateEngine 의 큰 역할은 3가지다.
    - Creating an instance of TemplateEngine
        - TemplateEngine 의 인스턴스 생성 비용은 비싸므로 하나의 인스턴스를 만들고 재사용하기를 권장한다.
        - **인스턴스를 만드는 비용이 비싸다라고 하는건 뭘까? 생성자 쪽을 일단 봐보자.**
            - 비용이 비싸다라는 말은 final 변수로 생성하는 인스턴스가 굉장히 많다라는 뜻이다. 하나의 인스턴스를 만들기 위해서 많은 인스턴스들을 생성해야 하니까 많은 메모리를 사용하게 되니까 비용이 비싸다.
            - ICacheManager 인터페이스의 구현체인 StandardCacheManager 객체를 만든다.
            - IEngineContextFactory 인터페이스의 구현체인 StandardEngineContextFactory 객체를 구현한다.
            - IMessageResolver 인터페이스의 구현체인 StandardMessageResolver 객체를 만든다.
            - ILinkBuilder 인터페이스의 구현체인 StandardLinkBuilder 객체를 만든다.
            - IDecoupledTemplateLogicResolver 인터페이스의 구현체인 StandardDecoupledTemplateLogicResolver 객체를 만든다.
            - IDialect 인터페이스의 구현체인 StandardDialect 객체를 만든다.

    - Configuring the TemplateEngine
        - TemplateResovler 를 추가하고 싶다면 addTemplateResolver 메소드를 통해서 할 수 있다.
        - 사용할 TemplateResolver 가 많다면 setTemplateResolvers 메소드를 통해서 가능하다.
        - 단 하나의 TemplateResolver 를 사용한다면 setTemplateResolver 메소드를 이용하면 된다.
        - TemplateResolver 를 세팅하지 않는다면 기본적으로  StringTemplateResolver 를 사용한다.
        - Template 에는 외부 메시지들 (국제화를 위한 메시지들) 을 넣어주는게 가능한데 이는 MessgaeResolver 를 통해서 가능하다. 이것도 세팅하지 않는다면 StandardMessageResolver 를 기본으로 사용한다.

    - TemplateExecution
        - Template 을 실행하는 방법으로 process() 메소드를 지원한다. 근데 이 메소드는 템플릿 처리 결과를 String 으로 반환받는 방법과 Argument 로 전달받은 Writer 를 통해서 쓰는 방법이 있는데 Writer 방법을 추천한다.
        - Writer 를 이용하는걸 추천한다.
            - 전체 결과를 가지는 String 을 생성하는 것보다 생성되는 즉시 Output Stream 으로 쓰는걸 원한다는듯.

### TemplateManager

- TemplateEngine 의 process() 메소드에서 TemplateManager.parseAndProcess() 메소드를 호출한다.
- TemplateManager 가 각각의 Parser 들을 가지고 있네.
    - HtmlParser
    - XmlParser 등등
- this.configuration 에 TemplateResolver 를 가지고 있다. 여기서는 SpringResourceTemplateResolver

### TemplateResolver

- Template 을 읽고 TemplateEngine 에게 가져오는 역할을 해준다.
- TemplateManager.resolveTemplate 메소드 내에있는  TemplateResolve.resolveTemplate() 메소드를 통해서 TemplateResolution 을 가지고온다.

### TemplateResolution

- TemplateResolver 에 의해 만들어진 객체로 Template 이 resolve 될 때 생긴다.
- 담긴 정보는 Template 리소스를 나타내는 정보와 Template Mode 에 관한 정보, 캐시에 적용할 수 있는 정보들이 포함된다.
    - resolve 뜻이 뭔데? 어떠한 Problem 에 대한 해결책을 찾았다. 라는 뜻
    - TemplateResolver 가 TemplateResolution 을 반환했다고 해서 Template 실제로 존재하는 건 아니다. ITemplateResolver 의 구현에 따라 다르다. 왜냐하면 존재하는지 체크하기, 실제로 가져오기 로 인한 두번의 Access 를 하니까 Performance 이슈가 생길 수 있다. 그래서 어떠한 구현체들은 존재하는지 체크를 안하는 경우도 있다.
    - 엑세스 하는 방법은 어떻게 되는데? I/O 기반인가? ㅇㅇ
        - Spring Core Class 인 Resource 인터페이스의 구현체인 ClassPathResource 의 exist() 메소드를 통해서 해결한다.
        - 더 들어가면 java.lang 패키지에 있는 classLoader.getResource() 메소드와 ClassLoader.getSystemResource() 를 통해서 체크한다.

### TemplateData

- 현재 처리할 Template 에 관한 모든 데이터가 들어있는 객체

### Handler

- parsing 하고 각 Tag 에 맞는 데이터를 찾는다.

### HTMLTemplateParser

### SpringTemplateEngine

### TemplateResource

### ITemplateHandler

### ITemplateParser

### ICache (TemplateCache)

### ViewResolver

- 역할: 뷰 이름을 가지고 실제 뷰 파일을 만들어주는 역할을 해준다.

### ContentNegotiationViewResolver

- 여러가지 ViewResolver 들을 가지고 있고 이들 중에서 뷰 파일을 생성해줄 수 있는 최적의 뷰를 찾아준다.
- 가지고 있는 뷰 리졸버는 다음과 같다.
    - BeanNameViewResolver
    - ThymeleafViewResolver
    - InternalResourceViewResolver
    - InternalResourceViewResolver
- 최적의 뷰를 찾을때 mediaType 을 이용하는건가? 각 지원하는 MediaType 별로 뷰 가 선택되는건가?
- 

### ThymeleafViewResolver

### InternalResourceViewResolver

### View

### ModelAndView

- View 와 Model 을 모두 가지고 있는 것

### ThymeleafView

### InternalResourceView

### MergeModel

- templateStaticVariables
- pathVariableSelector
- Model
- 여러가지 모델들을 합친 MergeModel 이 WebExpressionContext 객체를 생성하는데 사용된다.

### WebExpressionContext

### Writer

- Writer 의 사용 목적은 뭔데?

## 정리

### Document

- i.e 뜻은 "That is to say" 라는 말이다. "즉 다시 말해서"
- e.g 뜻은 "for example" 이라는 뜻이다.
- [i.g](http://i.ge) 와 e.g 둘 다 라틴어를 말한다.

### Dictionary

- Template
- Resolve
    - settle or find a solution

### Concurrency

- Two-Checked Mechanism

```java
// AbstractCacheManager 
if (!this.templateCacheInitialized) {
	synchronized(this) {
		// 한번 더 검사하는 로직. 
		if (!this.templateCacheInitialized) {
			this.templateCache = this.initializeTemplateCache(); 
			this.templateCacheInitialized = true;
		} 
	}
} 
```

### OOP

- Method 이름이 같은 것이 보인다.
    - TemplateManager.resolveTemplate() 과 TemplateResolver.resolveTemplate() 이렇게.
    - 아무래도 실제적인 작업을 수행하는건 TemplateResolver 지만 부가적인 작업들을 합쳐서 추상화해서 한번 더 감싸는듯.

### Cache

- LRU 캐시 메커니즘을 보고 싶다면 StandardCache 객체의 구현을 보면 될 듯.
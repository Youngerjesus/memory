> __Spiring Security Framework 를 분석하고 이를 정리한 글입니다.__
> 
> __정리할 내용은 다음과 같습니다.__
> - Spring Security 란? 
> - Spring Security 사용방법 
> - Spring Security 의 초기화 과정
> - Spring Security Architecture
> - Spring Security 의 주요 FilterChain 들에 대해서 알아보자
>
> __References__
> - [Spring Security References](https://spring.io/projects/spring-security)
> - [Details of spring security initialization process](https://developpaper.com/details-of-spring-security-initialization-process/)
> - [Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture)
> - [Spring Security Essential Classes](https://docs.spring.io/spring-security/site/docs/3.2.8.RELEASE/apidocs/org/springframework/security/web/)

***

## Spring Security 란? 

스프링 시큐리티의 정의를 먼저 보자. 

레퍼런스에는 다음과 같이 적혀있다. 

> Spring Security is a powerful and highly customizable authentication and access-control framework. it is the de-facto standard for securing Spring-based applications.
> 
> Spring Security is a framework that focuses on providing both authentication and authorization to java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirement.  

정리하자면 스프링 시큐리티는 스프링 기반의 어플리케이션에서 인증과 인가 검사를 통해 리소스 접근 여부를 컨트롤 해줄 수 있는 프레임워크이다. 

핵심적인 특징은 Real World 프로젝트에서 사용자의 요구사항을 반영하기가 쉽다라는 점이다. 

***

## Spring Security 사용 방법 

스프링 시큐리티를 사용하는 방법은 간단하다. 

빌드툴 Maven 을 기준으로 `pom.xml` 에 다음과 같이 의존성을 넣어주면 스프링 부트의 자동설정 기능으로 인해 빈이 만들어지고 기본으로 사용하는 FilterChain 이 등록된다. 

```xml
<dependency>
 	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

이를 내 프로젝트에 맞게 커스텀하게 사용하고 싶다면 `WebSecurityConfigurerAdapter` 인터페이스를 구현한 `SecurityConfig` 클래스를 작성하면 된다. 

```java
@EnableWebSecurity 
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    	,,,
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception { 
    	,,,
    }
}
```
 
이 `SecurityConfig` 에서 커스터마이징하게 사용할 수 있도록 다양한 API 들을 제공해준다. 

핵심적인 API 들을 제공해주는 게 `HttpSecurity` 와 `WebSecurity` 가 있고 이들을 이용해서 설정하면 된다. 

`HttpSecurity` 는 인증과 인가에 관련된 세부 설정을 도와주는 API 를 제공해 주고 

`WebSecurity` 는 `ignoring()` 메소드를 통해 스프링 시큐리티 필터를 적용하지 않을 리소스를 지정해줄 수 있다.   

***

## Spring Security 의 초기화 과정 

`SecurityConfig` 파일에서 설정한 정보가 어떻게 리소스 접근에 대한 Access Control 을 해줄 수 있을까? 

어떻게 이런 마법같은 일이 일어나는지 한번 스프링 시큐리티 클래스의 코드를 보면서 파헤쳐보자. 

`SecurityConfig` 파일에 있는 `@EnableWebSecurity` 를 보면 다음과 같다. 

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import({ WebSecurityConfiguration.class, SpringWebMvcImportSelector.class, OAuth2ImportSelector.class,
		HttpSecurityConfiguration.class })
@EnableGlobalAuthentication
@Configuration
public @interface EnableWebSecurity {

	/**
	 * Controls debugging support for Spring Security. Default is false.
	 * @return if true, enables debug support with Spring Security
	 */
	boolean debug() default false;

}
```

여기서 눈여겨 볼 부분은 @Import 인데 `WebSecurityConfiguration.class` 라는 Configuration 설정 파일을 가지고 있다. 

이 설정 파일에서 스프링 시큐리티의 코어 클래스들을 빈으로 만드는 작업을 한다. 

실제로 `WebSecurityConfiguration.class` 파일을 가서 보면 다음과 같이 구성되어 있다. 

```java
@Configuration(proxyBeanMethods = false)
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {
	,..
}
```

이 설정 파일의 핵심은 __`FilterChainProxy` 를 만들고 이를 스프링 빈으로 등록시키는 것이다.__

`FilterChainProxy` 는 이후에도 자세하게 설명하겠지만 핵심적인 특징은 클라이언트에게서 들어온 요청을 서블릿 필터의 스펙을 만족하는 스프링 컨테이너에 등록된 빈에게 필터 처리를 하도록 하는 것이다. 

요청이 처리되는 전체 과정을 보면 이 말을 이해하기 쉽다. 

처음 클라이언트에서 온 요청은 WAS 가 먼저 맞이하는데 이때 서블릿 필터를 통과한다. 

서블릿 필터중에 `DelegatingFilterProxy` 라는 클래스가 있는데 이는 직접 필터 처리를 하는게 아니라 다른 클래스에게 필터 처리를 위임하는 역할을 한다. 

이 다른 클래스가 `FilterChainProxy` 이다. `DelegatingFilterProxy` 는 스프링 IoC 컨테이너로부터 `springSecurityFilterChain` 이라는 이름을 가진 빈을 찾고 그 필터에게 작업을 위임해준다. 

그리고 `FilterChainProxy` 는 각각의 목적에 맞는 여러개의 `Filter` 들을 체인 형태로 가지고 있고 요청들을 하나씩 `Filter` 들에게 적용시켜보면서 필터 처리를 해나가는 구조로 이뤄져있다. 

즉 한 필터에서 필터 처리를 한 후 다음 필터에게 전달하는 과정을 진행한다. 

`FilterChainProxy` 에 대해서 조금만 더 설명을 하자면 요청의 URI 를 보고 처리해줄 수 있는 필터들을 선택한다. 이는 `RequestMatcher` 를 통해서 이런 작업을 해준다. 

이제다시 `WebSecurityConfiguration.class` 가 어떻게 `FilterChainProxy` 를 만드는지 확인해보자.  

이 설정 클래스에서 만들어지는 빈을 보면 어디서 `FilterChainProxy` 가 만들어지는지 볼 수 있다. 

```java
@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)
public Filter springSecurityFilterChain() throws Exception {
    boolean hasConfigurers = this.webSecurityConfigurers != null && !this.webSecurityConfigurers.isEmpty();
    boolean hasFilterChain = !this.securityFilterChains.isEmpty();
    Assert.state(!(hasConfigurers && hasFilterChain), "Found WebSecurityConfigurerAdapter as well as SecurityFilterChain. Please select just one.");
    if (!hasConfigurers && !hasFilterChain) {
    	WebSecurityConfigurerAdapter adapter = this.objectObjectPostProcessor
    	.postProcess(new WebSecurityConfigurerAdapter() {});
    
        this.webSecurity.apply(adapter);
    }
    for (SecurityFilterChain securityFilterChain : this.securityFilterChains) {
    	this.webSecurity.addSecurityFilterChainBuilder(() -> securityFilterChain);
    	for (Filter filter : securityFilterChain.getFilters()) {
    		if (filter instanceof FilterSecurityInterceptor) {
    			this.webSecurity.securityInterceptor((FilterSecurityInterceptor) filter);
    			break;
    		}
    	}
    }
    
    for (WebSecurityCustomizer customizer : this.webSecurityCustomizers) {
    	customizer.customize(this.webSecurity);
    }
    
    return this.webSecurity.build();
}
```

이 빈을 만들어주는 메소드를 보면 결국 `WebSecurity` 클래스의 `build()` 메소드를 통해서 `springSecurityFilterChain` 이 만들어지는 걸 알 수 있다. 

그렇다면 `WebSecurity` 클래스는 어디서 처음 만들어질까? 찾아보면 이 설정 클래스에서 `setFilterChainProxySecurityConfigurer` 이라는 메소드에서 만들어진다. 

```java
@Autowired(required = false)
public void setFilterChainProxySecurityConfigurer(ObjectPostProcessor<Object> objectPostProcessor,
			@Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}") List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers)
			throws Exception {
		
    this.webSecurity = objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));
    if (this.debugEnabled != null) {
    	this.webSecurity.debug(this.debugEnabled);
    }
	
    webSecurityConfigurers.sort(AnnotationAwareOrderComparator.INSTANCE);
    Integer previousOrder = null;
    Object previousConfig = null;
    for (SecurityConfigurer<Filter, WebSecurity> config : webSecurityConfigurers) {
    	Integer order = AnnotationAwareOrderComparator.lookupOrder(config);
        if (previousOrder != null && previousOrder.equals(order)) {
        	throw new IllegalStateException("@Order on WebSecurityConfigurers must be unique. Order of " + order + " was already used on " + previousConfig + ", so it cannot be used on " + config + " too.");
        }
        
        previousOrder = order;
        previousConfig = config;
     }
	
    for (SecurityConfigurer<Filter, WebSecurity> webSecurityConfigurer : webSecurityConfigurers) {
    	this.webSecurity.apply(webSecurityConfigurer);
     }
	
    this.webSecurityConfigurers = webSecurityConfigurers;
 }
```
이 메소드를 보면 `WebSecurity` 클래스를 만들기 위해서 필요한 클래스가 두 개가 있는데 `objectPostProcessor` 와 `List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers` 가 필요하다. 

여기서 `webSecurityConfigurers` 에 우리가 이전에 스프링 시큐리티 커스터마이징 하기 위해 만들었던 `SecurityConfig` 클래스가 들어있다.

어떻게 이것들을 가져올 수 있었을까? 메소드 앞에있던`@Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}")` 에 주목하면 되는데 

저기에 있는 메소드의 구현체를 보면 `BeanFactory` 를 통해서 우리가 만든 `SecurityConfig` 를 타입으로 조회해서 가지고 온다. 

`SecurityConfig` 는 인터페이스인 `WebSecurityConfigurerAdapter` 를 상속받는데 이 `WebSecurityConfigurerAdapter` 는 Adapter 패턴으로 `WebSecurityConfigurer` 와 연결되어 있다. 

이렇게 우리가 설정한 `SecurityConfig` 클래스와 `WebSecurityConfigurer` 는 연결되어있고 이를 통해 `WebSecurityConfigurer` 타입으로 조회하면 `SecurityConfig` 클래스가 조회되는 것이다. 

즉 `BeanFactory` 에 우리가 만든 설정 파일을 가져오고 그 정보를 바탕으로 `springSecurityFilterChain` 을 만드는 것이다. 

여기선 너무 길어서 설명하지 않겠지만 `springSecurityFilterChain` 이 만들어지는 과정도 굉장히 흥미롭다. 

특히 재밌었던 부분은 `FilterChain` 이 딱 한번만 생성되기 위해서 `AtomicVariable` 을 통해서 만드는 것과 `HttpSecurity` 클래스를 이용해 16 개의 필터체인을 만드는 부분이다.

***

## Spring Security Architecture

Application 에서 보안과 관련된 부분은 결국 Authentication (인증) 과 Authorization (인가) 이다. 

간단하게 둘의 개념을 살펴보면 인증은 "리소스를 요청하는 대상이 누군인지 검사하는 것" 이라면 인가는 "너가 누군지 알겠는데, 너가 권한이 있어?" 를 물어보는 과정이다. (때떄로 사람들은 인가를 다른 의미로 Access Control 이라고 부르기도 한다.) 

Spring Security 에서 이런 인증과 인가를 구성하는 핵심 컴포넌트들과 이들의 관계를 살펴보자. 

### Authentication (인증)

인증을 담당하는 핵심 인터페이스는 `AuthenticationManager` 이다. 

```java
/**
 * Processes an {@link Authentication} request.
 *
 * @author Ben Alex
 */
public interface AuthenticationManager {

	/**
	 * Attempts to authenticate the passed {@link Authentication} object, returning a
	 * fully populated <code>Authentication</code> object (including granted authorities)
	 * if successful.
	 * <p>
	 * An <code>AuthenticationManager</code> must honour the following contract concerning
	 * exceptions:
	 * <ul>
	 * <li>A {@link DisabledException} must be thrown if an account is disabled and the
	 * <code>AuthenticationManager</code> can test for this state.</li>
	 * <li>A {@link LockedException} must be thrown if an account is locked and the
	 * <code>AuthenticationManager</code> can test for account locking.</li>
	 * <li>A {@link BadCredentialsException} must be thrown if incorrect credentials are
	 * presented. Whilst the above exceptions are optional, an
	 * <code>AuthenticationManager</code> must <B>always</B> test credentials.</li>
	 * </ul>
	 * Exceptions should be tested for and if applicable thrown in the order expressed
	 * above (i.e. if an account is disabled or locked, the authentication request is
	 * immediately rejected and the credentials testing process is not performed). This
	 * prevents credentials being tested against disabled or locked accounts.
	 * @param authentication the authentication request object
	 * @return a fully authenticated object including credentials
	 * @throws AuthenticationException if authentication fails
	 */
	Authentication authenticate(Authentication authentication) throws AuthenticationException;

}

```

`AuthenticationManager` 는 `authenticate()` 라는 메소드를 통해서 인증을 수행하는데 크게 3가지 기능을 가진다. 

- 인증을 성공하면 `Authentication` 객체를 반환하는 것. 여기서 인증을 성공했다는 뜻으로 `Authentication` 객체는 `authenticated` 속성이 true 로 들어간다. 

- 인증에 실패를하면 `AuthenticationException` 을 발생시키는 것. 

- 인증을 지원하지 않는다면 `Authentication` 객체가 아닌 null 객체를 반환하는 것. 

인증에 성공해서 만들어진 `Authentication` 객체는 인증 객체를 말하고 다음과 같은 정보를 포함한다. 

- `principle` : 사용자의 ID 혹은 User 객체를 포함한다. 여기서 User 객체는 `UserDetails` 를 말하고 User 의 일반적인 정보를 가지고 있는 클래스를 말한다. 일반적으로 휴대폰 번호나 이메일 유저 ID 등을 가지고 있는 객체다. 

- `credentials` : 사용자의 비밀번호 같은 걸 말한다. 인증에 성공하고 난 뒤에는 이 값이 필요가 없으므로 null 값을 넣도록 한다.

- `authorities` : 인증된 사용자의 권한 목록들을 말한다. 

- `details` : 인증 사용자의 부가 정보를 말한다. 

- `authenticated` : 인증 여부를 말한다. 

`Authentication` 객체는 `SecurityContext` 가 가지도록 구성되어 있고 `SecurityContext` 는 `SecurityContextHolder` 가 보관한다. 

`SecurityContextHolder` 는 `SecurityContext` 를 다양한 전략으로 이를 소유할 수 있는데 기본 전략은 Thread Local 에다가 저장이 된다라는 것이다.

이를 통해 같은 스레드 즉 하나의 요청에서 어디서든지 `SecurityContextHolder` 를 참조할 수 있다. (전역 참조가 가능해진다.) 

그렇다면 인증이 끝난 후 매 요청마다 스레드에 `SecurityContextHolder` 를 넣어주는 작업은 어떻게 할 수 있을까? 

일단 인증을 매번 하면 안되니까 인증에 성공한 후 만들어지는`SecurityContext` 를 다른 저장소에 넣어주는 작업이 필요하다. 

이런 저장소 역할을 해주는게 `SecurityContextRepository` 인터페이스 이고 다음과 같다.

```java
/**
 * Strategy used for persisting a {@link SecurityContext} between requests.
 * <p>
 * Used by {@link SecurityContextPersistenceFilter} to obtain the context which should be
 * used for the current thread of execution and to store the context once it has been
 * removed from thread-local storage and the request has completed.
 * <p>
 * The persistence mechanism used will depend on the implementation, but most commonly the
 * <tt>HttpSession</tt> will be used to store the context.
 *
 * @author Luke Taylor
 * @since 3.0
 * @see SecurityContextPersistenceFilter
 * @see HttpSessionSecurityContextRepository
 * @see SaveContextOnUpdateOrErrorResponseWrapper
 */
public interface SecurityContextRepository {

	/**
	 * Obtains the security context for the supplied request. For an unauthenticated user,
	 * an empty context implementation should be returned. This method should not return
	 * null.
	 * <p>
	 * The use of the <tt>HttpRequestResponseHolder</tt> parameter allows implementations
	 * to return wrapped versions of the request or response (or both), allowing them to
	 * access implementation-specific state for the request. The values obtained from the
	 * holder will be passed on to the filter chain and also to the <tt>saveContext</tt>
	 * method when it is finally called. Implementations may wish to return a subclass of
	 * {@link SaveContextOnUpdateOrErrorResponseWrapper} as the response object, which
	 * guarantees that the context is persisted when an error or redirect occurs.
	 * @param requestResponseHolder holder for the current request and response for which
	 * the context should be loaded.
	 * @return The security context which should be used for the current request, never
	 * null.
	 */
	SecurityContext loadContext(HttpRequestResponseHolder requestResponseHolder);

	/**
	 * Stores the security context on completion of a request.
	 * @param context the non-null context which was obtained from the holder.
	 * @param request
	 * @param response
	 */
	void saveContext(SecurityContext context, HttpServletRequest request, HttpServletResponse response);

	/**
	 * Allows the repository to be queried as to whether it contains a security context
	 * for the current request.
	 * @param request the current request
	 * @return true if a context is found for the request, false otherwise
	 */
	boolean containsContext(HttpServletRequest request);

}
```

Spring Security 에서 `SecurityContextRepository` 의 기본 구현체로 사용하고 있는게 `HttpSessionSecurityContextRepository` 이고 `SecurityContext` 를 `HttpSession` 에 저장한다.

즉 `HttpSessionSecurityContextRepository` 는 `SecurityContext` 를 저장하는 저장소로 `HttpSession` 을 사용한다. 

그래서 인증을 하고나서 다음 요청을 하면 `HttpSession` 에서 `SecurityContext` 를 가지고 와서 `SecurityContextHolder` 에 넣어주면 되므로 번거로운 인증 작업을 하지 않는다.

세션에서 `SecurityContext` 를 찾고싶을 땐 쿠키에 저장되어있는 `JSESSIONID` 를 기본적으로 사용한다. 

이전에 인증을 헀는지 체크하고 인증을 한 적이 있다면 빠르게 시큐리티 저장소에서 `SecurityContext` 를 조회하도록 해주는 필터가 있는데 이게 `SecurityContextPersistentFiler` 이다. 

이 필터도 마찬가지로 `FilterChainProxy` 에 등록된 필터중에 하나다. 이후에 `FilterChainProxy` 가 가지고 있는 여러 필터들에 대해서 알아보자. 

__이어서 다시 `AuthenticationManager` 에 대해서 얘기해보자.__

`AuthenticationManager` 는 실제로 인증 처리 자체의 작업은 `AuthenticationProvider` 에게 위임한다. 

`AuthenticationProvider` 의 인터페이스는 다음과 같다.

```java
public interface AuthenticationProvider {

	/**
	 * Performs authentication with the same contract as
	 * {@link org.springframework.security.authentication.AuthenticationManager#authenticate(Authentication)}
	 * .
	 * @param authentication the authentication request object.
	 * @return a fully authenticated object including credentials. May return
	 * <code>null</code> if the <code>AuthenticationProvider</code> is unable to support
	 * authentication of the passed <code>Authentication</code> object. In such a case,
	 * the next <code>AuthenticationProvider</code> that supports the presented
	 * <code>Authentication</code> class will be tried.
	 * @throws AuthenticationException if authentication fails.
	 */
	Authentication authenticate(Authentication authentication) throws AuthenticationException;

	/**
	 * Returns <code>true</code> if this <Code>AuthenticationProvider</code> supports the
	 * indicated <Code>Authentication</code> object.
	 * <p>
	 * Returning <code>true</code> does not guarantee an
	 * <code>AuthenticationProvider</code> will be able to authenticate the presented
	 * instance of the <code>Authentication</code> class. It simply indicates it can
	 * support closer evaluation of it. An <code>AuthenticationProvider</code> can still
	 * return <code>null</code> from the {@link #authenticate(Authentication)} method to
	 * indicate another <code>AuthenticationProvider</code> should be tried.
	 * </p>
	 * <p>
	 * Selection of an <code>AuthenticationProvider</code> capable of performing
	 * authentication is conducted at runtime the <code>ProviderManager</code>.
	 * </p>
	 * @param authentication
	 * @return <code>true</code> if the implementation can more closely evaluate the
	 * <code>Authentication</code> class presented
	 */
	boolean supports(Class<?> authentication);

}
```

`AuthenticationProvider` 인터페이스를 보면 두 가지 메소드밖에 없어서 이해하기 쉬운데  

`supports()` 메소드는 파라미터로 받는 `Authentication` 객체를 보고 인증 처리를 해줄 수 있는지 판단하는 메소드이고 

`authenticate()` 메소드가 실제적인 인증 처리 작업을 수행해주는 메소드다. 클라이언트의 인증 요청을 통해 만들어진 `Authentication` 객체를 보고 유효성을 판단한 후 새롭게 `Authentication` 객체를 만드는 작업을 수행한다.

__`AuthenticationProvider` 가 유효성을 판단하기 위해 하는 작업은 결국 결국에 비교하는 작업이다.__

클라이언트가 던진 인증 정보와 실제로 우리가 보관하고 있는 인증 정보를 비교하는 작업이 필요한데 이는 `UserDetailsService` 인터페이스의 `loadUserByUsername()` 이라는 메소드를 통해서 가능하다. 

`UserDetailsService` 인터페이스의 역할은 username 을 바탕으로 실제 우리가 가지고 있는 유저 정보를 가지고 오는 역할을 해준다. (예를들면 데이터베이스 같은 곳에서 가지고 올 수 있겠다.) 

이렇게 `AuthenticationProvider` 를 통해서 인증 검사를 진행한다. 

`AuthenticationManager` 는 인증 처리 해주는 방법에 따라 `AuthenticationProvider` 를 여러개 가지고 있는데

하나씩 `AuthenticationProvider` 를 꺼내보면서 인증 처리해줄 수 있는지 묻고 그렇다면 인증 처리를 하는 구조로 이뤄져있다. 

이런 작업들은 `AuthenticationManager` 의 기본 구현체인 `ProviderManager` 를 보면 알 수 있는데 다음과 같다.

```java
// ProviderManager 의 authenticate() 메소드. 
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Class<? extends Authentication> toTest = authentication.getClass();
    AuthenticationException lastException = null;
    AuthenticationException parentException = null;
    Authentication result = null;
    Authentication parentResult = null;
    int currentPosition = 0;
    int size = this.providers.size();
    
    for (AuthenticationProvider provider : getProviders()) {
    	if (!provider.supports(toTest)) {
        	continue;
        }
        if (logger.isTraceEnabled()) {
        	logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
                provider.getClass().getSimpleName(), ++currentPosition, size));
        }
			
        try {
        	result = provider.authenticate(authentication);
            if (result != null) {
            	copyDetails(authentication, result);
                break;
            }
        }
        catch (AccountStatusException | InternalAuthenticationServiceException ex) {
            prepareException(ex, authentication);
            // SEC-546: Avoid polling additional providers if auth failure is due to
            // invalid account status
            throw ex;
        }
        catch (AuthenticationException ex) {
        	lastException = ex;
        }
    }
		
    if (result == null && this.parent != null) {
        // Allow the parent to try.
        try {
            parentResult = this.parent.authenticate(authentication);
            result = parentResult;
        }
        catch (ProviderNotFoundException ex) {
        // ignore as we will throw below if no other exception occurred prior to
        // calling parent and the parent
        // may throw ProviderNotFound even though a provider in the child already
        // handled the request
        }
        catch (AuthenticationException ex) {
            parentException = ex;
            lastException = ex;
        }
    }
    
    if (result != null) {
        if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
        // Authentication is complete. Remove credentials and other secret data
        // from authentication
            ((CredentialsContainer) result).eraseCredentials();
    	}
        // If the parent AuthenticationManager was attempted and successful then it
        // will publish an AuthenticationSuccessEvent
        // This check prevents a duplicate AuthenticationSuccessEvent if the parent
        // AuthenticationManager already published it
        if (parentResult == null) {
            this.eventPublisher.publishAuthenticationSuccess(result);
        }
     	return result;
    }

    // Parent was null, or didn't authenticate (or throw an exception).
    if (lastException == null) {
        lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
            new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
    }
		
    // If the parent AuthenticationManager was attempted and failed then it will
    // publish an AbstractAuthenticationFailureEvent
    // This check prevents a duplicate AbstractAuthenticationFailureEvent if the
    // parent AuthenticationManager already published it
    if (parentException == null) {
    	prepareException(lastException, authentication);
    }
    
    throw lastException;
}
```

`ProviderManager` 의 인증 처리 작업은 좀 특이한데 가지고 있는 `AuthenticationProvider` 가 모두 인증 처리를 수행할 수 없다면 `parent` 에게 인증 처리 작업을 요청한다. 

`ProviderManager` 는 부모 - 자식 관계로 이뤄져있고 부모 객체도 `ProviderManager` 이다. 

왜 이렇게 구성되어있는지 좀 궁금했는데 레퍼런스에서는 `logical groups` 를 만들기 위해서 이렇게 구성했다고 한다.

어플리케이션이 여러 URI 별로 인증 정책을 다르게 가져가고 싶을 때, 각 URI 마다 `ProviderManager` 를 두고 각 `AuthenticationProvider` 가 처리하지 못한다면 부모의 `ProviderManager` 에게 위임하는 식으로 구조가 잡혀있다. 

### Authorization (인가) 

인증 검사를 통과하고 나면 인가 검사를 한다. 

인가 검사는 __리소스에 접근할 권한이 있는지__ 검사한다고 생각하면 된다. 

스프링 시큐리티가 지원하는 권한 계층은 크게 다음과 같이 3 가지가 있는데 여기선 웹 계층만 보겠다. 

- 웹 계층: URL 요청에 따른 보안

- 서비스 계층: 메소드 단위의 보안

- 도메인 계층: 객체 단위의 보안 

인가 검사는 마지막 `SecurityFilterChain` 인 `FilterSecurityInterceptor` 에서 이뤄진다. 

인가 체크를 하기 전에 먼저 인증 여부를 확인하는데 인증 객체 자체가 없다면 `AuthenticationException` 이 발생하게 된다. 

인증 객체가 있다면 요청하는 리소스가 가지고 있는 권한을 조회해본다. 

리소스가 가지고 있는 권한 즉 "리소스를 요청할려면 이 권한은 있어야한다." 라는 정보를 가지고 있는 클래스는 `SecurityMetadataSource` 인터페이스다. 

이 클래스로부터 리소스의 권한을 조회해보고 null 인 경우 즉 아무런 권한이 없어도 되는 경우라면 인가 검사에 통과하게 된다. 

권한이 있는 경우라면 권한 검사를 받게 되는데 이런 검사를 해주는게 `AccessDecisionManager` 이다. 

`AccessDecisionManager` 의 인터페이스는 다음과 같다. 

```java
public interface AccessDecisionManager {

	/**
	 * Resolves an access control decision for the passed parameters.
	 * @param authentication the caller invoking the method (not null)
	 * @param object the secured object being called
	 * @param configAttributes the configuration attributes associated with the secured
	 * object being invoked
	 * @throws AccessDeniedException if access is denied as the authentication does not
	 * hold a required authority or ACL privilege
	 * @throws InsufficientAuthenticationException if access is denied as the
	 * authentication does not provide a sufficient level of trust
	 */
	void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
			throws AccessDeniedException, InsufficientAuthenticationException;

	/**
	 * Indicates whether this <code>AccessDecisionManager</code> is able to process
	 * authorization requests presented with the passed <code>ConfigAttribute</code>.
	 * <p>
	 * This allows the <code>AbstractSecurityInterceptor</code> to check every
	 * configuration attribute can be consumed by the configured
	 * <code>AccessDecisionManager</code> and/or <code>RunAsManager</code> and/or
	 * <code>AfterInvocationManager</code>.
	 * </p>
	 * @param attribute a configuration attribute that has been configured against the
	 * <code>AbstractSecurityInterceptor</code>
	 * @return true if this <code>AccessDecisionManager</code> can support the passed
	 * configuration attribute
	 */
	boolean supports(ConfigAttribute attribute);

	/**
	 * Indicates whether the <code>AccessDecisionManager</code> implementation is able to
	 * provide access control decisions for the indicated secured object type.
	 * @param clazz the class that is being queried
	 * @return <code>true</code> if the implementation can process the indicated class
	 */
	boolean supports(Class<?> clazz);

}
```

`AccessDecisionManager` 는 `decide()` 메소드를 통해서 인가 검사를 실행한다. 

실제적인 인가 처리는 `AccessDecisionVoter` 이 수행하고 `AccessDecisionManager` 는 인가 검사의 전략을 구성한다. 

인가 검사의 전략은 다음과 같다. 

- `AffirmativeBased` 는 여러 `AccessDecisionVoter` 들 중 인가 검사 하나만 통과해도 리소스에 접근할 수 있다. 

- `ConsensusBased` 는 민주주의 식으로 진행되며 다수표를 통해 리소스 접근을 허락할지 거부할지 결정한다. 동률인 경우 기본적으로 접근을 허용하며 이는 `allowIfEqualGrantedDeniedDecision()` 메소드를 통해서 변경하는게 가능하다. 

- `UnanimousBased` 는 만장일치를 통해 인가 검사를 진행한다. 

실제적인 인가 처리를 담당하는 `AccessDecisionVoter` 의 인터페이스는 다음과 같다.

```
public interface AccessDecisionVoter<S> {

	int ACCESS_GRANTED = 1;

	int ACCESS_ABSTAIN = 0;

	int ACCESS_DENIED = -1;

	/**
	 * Indicates whether this {@code AccessDecisionVoter} is able to vote on the passed
	 * {@code ConfigAttribute}.
	 * <p>
	 * This allows the {@code AbstractSecurityInterceptor} to check every configuration
	 * attribute can be consumed by the configured {@code AccessDecisionManager} and/or
	 * {@code RunAsManager} and/or {@code AfterInvocationManager}.
	 * @param attribute a configuration attribute that has been configured against the
	 * {@code AbstractSecurityInterceptor}
	 * @return true if this {@code AccessDecisionVoter} can support the passed
	 * configuration attribute
	 */
	boolean supports(ConfigAttribute attribute);

	/**
	 * Indicates whether the {@code AccessDecisionVoter} implementation is able to provide
	 * access control votes for the indicated secured object type.
	 * @param clazz the class that is being queried
	 * @return true if the implementation can process the indicated class
	 */
	boolean supports(Class<?> clazz);

	/**
	 * Indicates whether or not access is granted.
	 * <p>
	 * The decision must be affirmative ({@code ACCESS_GRANTED}), negative (
	 * {@code ACCESS_DENIED}) or the {@code AccessDecisionVoter} can abstain (
	 * {@code ACCESS_ABSTAIN}) from voting. Under no circumstances should implementing
	 * classes return any other value. If a weighting of results is desired, this should
	 * be handled in a custom
	 * {@link org.springframework.security.access.AccessDecisionManager} instead.
	 * <p>
	 * Unless an {@code AccessDecisionVoter} is specifically intended to vote on an access
	 * control decision due to a passed method invocation or configuration attribute
	 * parameter, it must return {@code ACCESS_ABSTAIN}. This prevents the coordinating
	 * {@code AccessDecisionManager} from counting votes from those
	 * {@code AccessDecisionVoter}s without a legitimate interest in the access control
	 * decision.
	 * <p>
	 * Whilst the secured object (such as a {@code MethodInvocation}) is passed as a
	 * parameter to maximise flexibility in making access control decisions, implementing
	 * classes should not modify it or cause the represented invocation to take place (for
	 * example, by calling {@code MethodInvocation.proceed()}).
	 * @param authentication the caller making the invocation
	 * @param object the secured object being invoked
	 * @param attributes the configuration attributes associated with the secured object
	 * @return either {@link #ACCESS_GRANTED}, {@link #ACCESS_ABSTAIN} or
	 * {@link #ACCESS_DENIED}
	 */
	int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);

}
```

`AccessDecisionVoter` 는 파라미터로 전달받은 `ConfigAttribute` 와  `Authentication` 객체에 있는 인가들을 비교하는 작업을 통해 인가 검사를 진행한다. (`Authentication` 객체의 `getAuthorities()` 메소드를 사용하면 권한들을 모두 가져올 수 있다.) 

`ConfigAttribute` 는 접근하는 리소스가 가지고 있는 권한에 대한 정보로 예를들면 `permitAll`, `authenticated`, `hasRole('ROLE_USER)` 과 같은 정보가 있다. 차례대로 모두 허락, 인증만 있으면 됨, 유저 권한이 있어야 됨을 의미한다. 

이렇게 인가 검사를 진행하고 리소스에 접근할 적절한 권한이 없다면 `AccessDeniedException` 이 발생하게 된다. 

***

## Spring Security 의 주요 FilterChain 들에 대해서 알아보자. 

여기선 `FilterChainProxy` 가 가지고 있는 필터들에 대해서 간략하게 알아볼건데 주 포인트는 어떤 역할을 해주는지 살펴볼 것이다.

자세한 정보는 레퍼런스를 통해 찾아보자. 

`FilterChainProxy` 가 가지고 있는 필터들은 다음과 같다. 

- 0번: `WebAsyncManagerIntegrationFilter`

- 1번: `SecurityContextPersistentFilter`

- 2번: `HeaderWriterFilter`

- 3번: `CsrfFilter`

- 4번: `LogoutFilter`

- 5번: `UsernamePasswordAuthenticationFilter`

- 6번: `DefaultLoginPageGeneratingFilter`

- 7번: `DefaultLogoutPageGeneratingFilter`

- 8번: `ConcurrencySessionFilter`

- 9번: `RequestCacheAwareFilter`

- 10번: `SecurityContextHolderAwareRequestFilter`

- 11번: `RememberMeAuthenticationFilter`

- 12번: `AnonymousAuthenticationFilter`

- 13번: `SessionManagementFilter`

- 14번: `ExceptionTranslationFilter`

- 15번: `FilterSecurityInterceptor`

### WebAsyncManagerIntegrationFilter 

이 필터는 Spring MVC 에서 `SecurityContext` 를 Async Handler 에서 사용할 수 있도록 지원하는 기능이다. 

예제로 보면 이해하기 쉬운데 다음과 같은 Async Handler 가 있다고 보자. 

```java
@GetMapping
public Callable<String> asyncHandler() {
    System.out.println(Thread.currentThread().getName());
    System.out.println(SecurityContextHolder.getContext().getAuthentication());
	return () -> {
        System.out.println(Thread.currentThread().getName());
        System.out.println(SecurityContextHolder.getContext().getAuthentication());
    	return "async handler"
    }
} 
```

원래 `SecurityContextHolder` 는 같은 Thread 내에서만 `SecurityContext` 를 공유한다. 

하지만 `WebAsyncManagerIntegrationFilter` 를 사용하면 Callable 내에있는 다른 스레드에서도 `SecurityContext` 를 참조할 수 있도록 해줄 수 있다. 

어떻게 이게 가능하냐면 요청을 비동기식으로 처리할 수 있게 지원해주는 `WebAsyncManager` 에 `interceptor` 로 `SecurityContextCallableProcessingInterceptor` 를 세팅해주기 때문이다. 

그래서 Callable 내에서 있는 코드를 실행하기 전에 `preProcess()` 라고 해서 `SecurityContext` 를 세팅해주고 processing 이 마무리 되면 `postProcess()` 를 호출해서 `SecurityContext` 를 정리해주는 일을 한다. (AOP 방식과 유사하다.) 

이런 방식을 통해 Async Handler 에서도 `SecurityContext` 를 참조하는게 가능해진다. 

추가로 궁금할 수 있는 문제가 있는데 다음과 같다. 

__Async Handler 가 아니라 Service 내에서 @Async 메소드를 호출하는 경우에는 어떻게 `SecurityContext`  에서 참조할 수 있을까?__ 

예제로 보면 다음과 같다. 

```java
@GetMaaping
public String handler() {
	return sampleService.asyncMerhod(); 
}
```

```java
@Service
public class sampleService {

    @Async
    public String asyncMethod() {
    	,,,
    }
}
```

```java
@EnableAsync
@SpringBootApplication
public class SpringBootBasicSecurityDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootBasicSecurityDemoApplication.class, args);
    }

}

```

이 경우에는 `SecurityContext` 를 감싸는 전략을 바꾸면 된다. 

기본적으로 `SecuriyContext` 를 감싸는 전략인 `MODE_THREADLOCAL` 에서 `MODE_INHERITABLETHREADLOCAL`  이걸로 바꾸면 된다. 

이 방식은 메인 스레드와 자식 스레드가 같은 `SecurityContext` 를 가지도록 하는 전략이다. 

바꾸는 메소드는 `SecurityContextHolder.setStrategy()` 메소드를 통해서 바꿀 수 있다. 

### SecurityContextPersistentFilter

이 필터는 한 번 인증 수행을 하고나면 다음번의 요청부터는 인증을 하지 않고 기존에 인증한 결과를 재사용 할 수 있도록 공유해주는 필터다. 

인증을 다시 하지 않기 위해서는 이를 어딘가에 저장해야 한다는 말인데 그 저장소로 `SecurityContextRepository` 에 저장한다. 

`SecurityContextRepository` 는 인터페이스이고 Spring Security 에서는 기본 구현체로 `HttpSessionSecurityContextRepository` 를 사용한다. 즉 `HttpSession` 에 `SecurityContext` 를 저장한다. 

### HeaderWriterFilter

우리가 직접 설정할 일이 거의 없는 필터중에 하나지만 이 필터가 하는 일은 조금 중요하다. 

이 필터의 역할을 HTTP 응답 헤더에 보안과 관련된 헤더 정보를 추가해주는 필터다. 

추가해주는 헤더 정보는 다음과 같다. 

- XContentTypeOptionsHeaderWriter
    
  - 마임 타입 스니핑이라는 어택을 방지해주는 역할을 해준다.

- XXssProtectionHeaderWriter

  - 브라우저에 내장된 XSS 필터를 적용해준다.

- CacheControlHeadersWriter
  
  - 정적인 요청 말고 동적인 요청에서 캐시를 적용하지 않도록 해서 캐시 히스토리 취약점을 방어해주는 필터를 적용해준다.

- HstsHeaderWriter
  
  - HTTPS 로만 소통하도록 강제해주는 역할을 한다.

- XFrameOptionsHeaderWriter
  
  - clickjacking 어택을 방지해주는 역할을 한다.

### CsrfFilter 

이 필터는 CSRF (Cross-site Request Forgery) 어택을 막아주는 필터다. 

CSRF 어택은 웹사이트에 접속하면 정상적인 웹사이트를 전달 받는게 아니라 공격자의 악의적인 웹페이지를 받게되고 의도치 않게 다른 API 요청을 날리게 하는 공격 기법이다. 

이를 막는 방법은 웹플리케이션에서 클라이언트에게 페이지를 줄 때 CSRF 토큰을 숨겨서 주는 것이다. 

그래서 요청이 오면 CSRF 토큰 값이 있는지 확인하는 필터를 적용하는 것인데 이를 통해 악의적인 페이지에서 오는 요청은 막아줄 수 있다. 

### LogoutFilter 

이 필터는 로그아웃을 처리해주는 필터다. 

코드로 보면 다음과 같다. 

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    throws IOException, ServletException {
    doFilter((HttpServletRequest) request, (HttpServletResponse) response, chain);
}

private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
    throws IOException, ServletException {
    if (requiresLogout(request, response)) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.format("Logging out [%s]", auth));
        }
        this.handler.logout(request, response, auth);
        this.logoutSuccessHandler.onLogoutSuccess(request, response, auth);
        return;
    }
    chain.doFilter(request, response);
}
```

크게 두 가지를 실행시켜주는데 `logoutHandler` 와 `logoutSuccessHandler` 를 실행시켜준다. 

`logoutHandler` 는 로그아웃 처리를 해주는 핸들러고 주로 세션을 무효화하고 `SecurityContext` 객체를 삭제하는 역할을 한다. 

`logoutSuccessHandler` 는 로그아웃이 성공하면 그 후에 처리하는 핸들러로 기본으로 사용되는 구현체는 설정한 URL 로 redirect 하는 일을 한다. 

`logoutHandler` 는 좀 특이한데 Composite Pattern 으로 이뤄져있다. 그래서 하나의 `logoutHandler` 를 실행하는 것 같지만 실제로는 여러개를 실행한다. 

### UsernamePasswordAuthenticationFilter 

formLogin 을 사용할 경우 인증 처리를 수행해주는 필터이다.

`AuthenticationManager` 의 구현체인 `ProviderManager` 가 인증 처리를 실행하고 `Authentication` 객체를 만들어주는 역할이 여기서 일어난다. 

### DefaultLoginPageGeneratingFilter 와 DefaultLogoutPageGeneratingFilter

이 필터는 기본 로그인 / 로그아웃 페이지를 만들어주는 역할을 하는 필터다. 

로그인 / 로그아웃 페이지로 이동할 때 이 필터가 작동한다.

### ConcurrencySessionFilter

매 요청마다 현재 사용자의 세션 만료 여부를 체크하는 필터다. 

세션이 만료되었을 경우 만료 처리를 수행하는데 주로 로그아웃 처리를 하던가 오류페이지로 이동하도록 처리한다. 

이 필터는 `SessionManagementFilter` 와 연계해서 주로 동작을 하는데 어떻게 동작하는 지는 다음 예시를 보자. 

1. 한 사용자가 인증을 하고 세션에 등록했다고 가정해보자.

2. 동일한 새로운 사용자가 와서 인증을 하면 `SessionManagementFilter` 가 처리를 한다. 이때 최대 허용 가능한 세션이 초과했을 경우에 동시 세션 제어 전략에 따라서 행동이 다르겠지만 기본 전략인 이전 사용자 세션 만료 전략에 따라서 즉시 이전 세션을 만료시킨다. (`sesion.expireNow()`)

3. 그 후에 처음 사용자가 요청을 하면 `ConcurrentSessionFilter` 에서 세션 만료 여부를 검사하고 만료되었다면 Logout 처리를 하던가 오류 페이지로 보낸다.

### RequestCacheAwareFilter

이 필터는 되게 간단하다. 코드는 다음과 같다. 

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
        HttpServletRequest wrappedSavedRequest = this.requestCache.getMatchingRequest((HttpServletRequest) request,
        (HttpServletResponse) response);
        
        chain.doFilter((wrappedSavedRequest != null) ? wrappedSavedRequest : request, response);
}
```
이 필터의 역할은 캐시된 요청이 있는지 확인하고 있다면 현재 요청이 아니라 캐시된 요청으로 처리를 할려고 하는 것이다. 

시나리오로 살펴보면 이 말을 이해하기 쉽다.

먼저 권한이 없는 상태에서 권한을 요구하는 어떤 페이지를 요청했다. 

그러면 `AccessDecisionManager` 에 의해 예외가 발생하게 되고 `ExceptionTranslationFilter` 에서 로그인 페이지로 리다이렉트 시키고 현재의 요청은 캐싱시킨다. 

그래서 이후에 인증 요청을 하면 캐싱된 요청 (요청 URI 가 변경된) 으로 바껴서 인증 처리를 수행하고 처음 요청했던 곳으로 이동한다. 

### SecurityContextHolderAwareRequestFilter

이 필터도 우리가 주로 설정하지 않아도 되는 필터 중 하나다. 

하는 일은 서블릿 스펙 3 를 지원해주는 일을 한다. 

### RememberMeAuthenticationFilter

이 필터는 `RememberMe` 인증을 수행해주는 필터다. 

`RememberMe` 는 세션이 만료되고 어플리케이션이 종료되도 어플리케이션이 사용자를 기억하는 기능으로 사용자의 쿠키를 통해 기억하는 기능이다.


### AnonymousAuthenticationFilter

이 필터는 `Authentication` 이 없는 경우 즉 인증 통과를 못한 유저의 경우 익명 사용자 라는 정보를 만들어 주는 역할을 한다. 

익명 사용자의 `Authentication` 구현체는 `AnonymousAuthenticationToken` 을 사용하고 세션에 저장할 필요는 없어서 저장하지 않는다. 

### SessionManagementFilter

이 필터는 세션과 관련된 여러가지 일들을 해준다. 하는 일은 다음과 같다. 

- 세션 관리

  - 인증 시 사용자의 세션정보를 등록, 조회, 삭제등의 세션 관리를 한다.

- 동시적 세션 제어

  - 동일 계정으로 접속할 때 허용되는 최대 세션 수를 설정하거나 전략을 설정할 수 있다.

- 세션 고정 보호

  - 인증할 때마다 새로운 세션 Id를 발급하도록 해서 공격자의 쿠키 조작을 방지할 수 있도록 한다.

- 세션 생성 정책
 
  - 다양한 세션 정책을 지원한다. (e.g Always, If_Required, Never, Stateless)

### ExceptionTranslationFilter

이 필터는 마지막 필터인 `FilterSecurityInterceptor` 가 던지는 예외를 처리해주는 필터다. 

즉 `FilterSecurityInterceptor` 와 연계해서 주로 처리를 하는데 실제 코드를 보면 이해하기 쉽다.

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws IOException, ServletException {
        try {
            chain.doFilter(request, response);
        }
        catch (IOException ex) {
            throw ex;
        }
        catch (Exception ex) {
            // Try to extract a SpringSecurityException from the stacktrace
            Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(ex);
            RuntimeException securityException = (AuthenticationException) this.throwableAnalyzer.getFirstThrowableOfType(AuthenticationException.class, causeChain);
            
            if (securityException == null) {
                securityException = (AccessDeniedException) this.throwableAnalyzer.getFirstThrowableOfType(AccessDeniedException.class, causeChain);
            }
            if (securityException == null) {
                rethrow(ex);
                }
            if (response.isCommitted()) {
                throw new ServletException("Unable to handle the Spring Security Exception " + "because the response is already committed.", ex);
            }
            handleSpringSecurityException(request, response, chain, securityException);
        }
}
```

즉 `ExceptionTranslationFilter` 가 `FilterSecurityInterceptor` 를 try-catch 로 감싸고 있는 구조로 이뤄져있다. 

주로 처리하는 예외는 `AuthenticationException` 와 `AccessDeniedException` 이다. 

`AuthenticationException` 예외는 인증 자체가 안되있는 예외로 로그인 페이지로 리다이렉트 시키고 현재 요청을 캐싱해놓는 처리를 해준다.

`AccessDeniedException` 는 권한이 없는 예외로 `AccessDeniedHandler` 에서 예외를 처리하도록 해줄 수 있다. 기본 처리는 403 에러를 보여주는 것이다. 

`AccessDeniedException` 에서 헷갈리기 쉬운 부분은 익명 사용자로 인증 객체가 있는 경우인데 이도 인증을 해야하므로 `AuthenticationException` 예외 처리와 똑같이 로그인 페이지로 리다이렉트 시키고 현재 요청은 캐싱한다. 

### FilterSecurityInterceptor

이 필터는 마지막에 위치한 필터로 인가 처리를 통해 리소스에 접근하는 요청에 승인 여부를 결정하는 필터다.

처리하는 방식은 인증 객체가 없이 이 필터 체인에 오면 `AuthenticationException` 이 발생하도록 하고 

인증 객체가 있지만 자원에 접근하는 권한이 없다면 `AccessDeniedException` 이 발생한다.

이렇게 발생한 예외는 `ExceptionTranslationFilter` 가 처리를 해준다. 
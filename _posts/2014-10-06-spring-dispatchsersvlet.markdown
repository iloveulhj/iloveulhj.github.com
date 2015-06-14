---
layout: post
title:  "[Spring] dispatcherServlet"
date:   2014-10-06 00:00:00
categories: posts spring
---

### Java의 서블릿입니다.

* FrameworkServlet, HttpServletBean, HttpServlet을 상속합니다.
![](/images/dispatcherservlet/dispatcherservlet1.jpg)
* 이미지 출처: [`http://www.javatpoint.com/life-cycle-of-a-servlet`](http://www.javatpoint.com/life-cycle-of-a-servlet)
* DispatcherServlet도 서블릿을 상속하였으므로 위 이미지의 생명주기를 가집니다.
* init()에서 필요한 bean들을 초기화하고 요청은 service()를 통해 처리합니다.

---

### Spring web MVC framework의 Front Controller입니다.

* Spring web MVC framewrok 중앙의 서블릿(Front Controller) 역할을 수행합니다.
![](/images/dispatcherservlet/dispatcherservlet2.png)
* 이미지 출처: [`spring.io`](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet)
* 요청에 대해 공통적인 작업을 먼저 수행한 후 적절한 controller에 위임하고 view를 만들어 응답해줍니다.

---

### 설정 파일 - WebApplicationContext

* DispatcherServlet는 각각의 WebApplicationContext를 가지고 있습니다.
![](/images/dispatcherservlet/dispatcherservlet3.gif)
* 이미지 출처: [`spring.io`](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet)
* WebApplicationContext의 정보를 통해 초기화를 하고 필요한 bean을 찾아 위임해줍니다.
* 기본 값은 XmlWebApplicationContext이며 다양한 형식을 지원합니다.  
[`http://docs.spring.io/spring/docs/current/javadoc-api/`](http://docs.spring.io/spring/docs/current/javadoc-api/)

---

# 초기화 - init()

* 서버(Servlet Container)가 시작하면 DispatcherServlet(Servlet)의 init()을 통해 초기화합니다.
* init()에서부터 호출되는 메서드 중에는 요청을 처리하고 뷰를 생성할 때 사용되는 특별한 bean들을 초기화하는 메서드도 존재합니다.
{% highlight java %}
protected void initStrategies(ApplicationContext context) {
	initMultipartResolver(context);
	initLocaleResolver(context);
	initThemeResolver(context);
	initHandlerMappings(context);
	initHandlerAdapters(context);
	initHandlerExceptionResolvers(context);
	initRequestToViewNameTranslator(context);
	initViewResolvers(context);
	initFlashMapManager(context);
}
{% endhighlight %}

* WebApplicationContext에 존재하는 특별한 bean들  

| Bean type | Explanation |  
| HandlerMapping | Maps incoming requests to handlers and a list of pre\- and post-processors (handler interceptors) based on some criteria the details of which vary by HandlerMapping implementation. The most popular implementation supports annotated controllers but other implementations exists as well. 요청에 위임할 컨트롤러를 결정하는 로직을 담당한다. |
| HandlerAdapter | Helps the DispatcherServlet to invoke a handler mapped to a request regardless of the handler is actually invoked. For example, invoking an annotated controller requires resolving various annotations. Thus the main purpose of a HandlerAdapter is to shield the DispatcherServlet from such details. 핸들러 매핑으로 선택한 컨트롤러를 호출할 때 사용하는 어댑터이다. |
| HandlerExceptionResolver | Maps exceptions to views also allowing for more complex exception handling code. 예외가 발생하였을 때 처리하는 로직을 담당한다. |
| ViewResolver | Resolves logical String-based view names to actual View types. 컨트롤러가 리턴한 뷰 이름을 참고해서 뷰 오브젝트를 찾아주는 로직을 담당한다. |
| LocaleResolver & LocaleContextResolver | Resolves the locale a client is using and possibly their time zone, in order to be able to offer internationalized views. 지역정보를 결정해주는 전략이다. |
| ThemeResolver | Resolves themes your web application can use, for example, to offer personalized layouts. 테마를 가지고 이를 변경해서 사이트를 구성할 경우 쓸 수 있는 테마 정보를 결정해주는 전략이다. |
| MultipartResolver | Parses multi-part requests for example to support processing file uploads from HTML forms.   |
| FlashMapManager | Stores and retrieves the "input" and the "output" FlashMap that can be used to pass attributes from one request to another, usually across a redirect.|

자료 출처 [`spring.io`](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet), `토비 스프링 3.1`

---

# 요청 처리 - service()

요청이 들어오면 service() - processRequest() - doService() 메서드가 호출됩니다.  

- processRequest() 전에 method type별로 분기할 수 있습니다.  
  
요청을 처리하는 과정  
  
- DispatcherServlet의 HTTP요청 접수
- DispatcherServlet에서 컨틀롤러로 HTTP 요청 위임
- 컨트롤러의 모델 생성과 정보 등록
- 컨트롤러의 결과 리턴: 모델과 뷰
- DispatcherServlet의 뷰 호출과 모델 참조
- HTTP응답 돌려주기  

출처: `토비의 스프링 3.1`
 
---
 
### doService()
{% highlight java %}
/* 간략하게 정리한 코드입니다. */
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
	// Make framework objects available to handlers and view objects.
	request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
	request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
	request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
	request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

	FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
	if (inputFlashMap != null) {
		request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
	}
	request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
	request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

	doDispatch(request, response);
}
{% endhighlight %}
* 요청을 처리하는데 필요한 Bean들을 request에 주입합니다.
* dispatch할 메서드를 호출합니다. 

---

### doDispatch()
{% highlight java %}
/* 간략하게 정리한 코드입니다. */
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	// Determine handler for the current request.
	mappedHandler = getHandler(processedRequest, false);

	// Determine handler adapter for the current request.
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

	if (!mappedHandler.applyPreHandle(processedRequest, response)) {
		return;
	}

	// Actually invoke the handler.
	// return ModelAndView
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

	applyDefaultViewName(request, mv);
	mappedHandler.applyPostHandle(processedRequest, response, mv);

	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);

	// Instead of postHandle and afterCompletion
	mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
	return;

	// Clean up any resources used by a multipart request.
	if (multipartRequestParsed) {
		cleanupMultipart(processedRequest);
	}
}	
{% endhighlight %}
* 핸들러와 어뎁터를 호출하고 여러 preHandle, handle, postHandle들을 통해 요청을 처리합니다.
* processDispatchResult를 타고 들어가면 아래와 같은 코드가 나옵니다.

{% highlight java %}
/* 간략하게 정리한 코드입니다. */
//processDispatchResult를 타고 들어가면 나오는 메서드입니다.
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
	// Determine locale for request and apply it to the response.
	Locale locale = this.localeResolver.resolveLocale(request);
	response.setLocale(locale);

	View view;
	if (mv.isReference()) {
		// We need to resolve the view name.
		view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
	}

	// Delegate to the View object for rendering.
	view.render(mv.getModelInternal(), request, response);
}
{% endhighlight %}
* render메서드를 통해 뷰를 만드는 작업을 합니다.

---

# 결론

* DispatcherServlet은 Servlet입니다.
* WebApplicationContext을 통해 Bean들을 등록하고 필요할 때 사용할 수 있습니다.
* 요청이 들어오면 DispatcherServlet은 특별한 Bean들을 통해 요청을 처리하고 뷰를 만듭니다.
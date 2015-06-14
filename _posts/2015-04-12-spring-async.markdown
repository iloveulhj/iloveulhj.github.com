---
layout: post
title:  "[Spring] @Async"
date:   2015-04-12 00:00:00
categories: posts spring
---

# Spring @Async

스프링에서는 메소드에 `@Async`를 붙임으로서 손 쉽게 비동기메소드를 생성 가능  
다시 말해 다른 thread를 통해 메소드를 실행할 수 있게 해줌  

---

# 실행 환경  
**TaskExecutor**  
java.util.concurrent.Executor과 같은 인터페이스  
쓰레드 풀을 이용할 때 java5에 대한 요구사항을 추상하하기 위해 이용  
spring에서 task namespace를 이용해 손쉽게 TaskExecutor bean을 등록할 수 있음  
{% highlight xml%}
<task:executor id="taskExecutor" pool-size="5"  />
{% endhighlight %}
pool-size외에도 다양한 옵션을 제공  
asynchronous mehtod를 실행하기 위해 TaskExecutor가 필요  

**@EnableAsync**  
asynchronous mehtod를 호출하는 빈에 `@EnableAsync`가 필요  
이 어노테이션은 스프링이 `@Async` 메서드를 백그라운드 쓰레드풀을 통해 실행하게 해줌  

---

**제약 사항**  
`@Async`는 public method에 붙여야 함  
self invoation의 경우(same class)는 동작하지 않음

---

# 구현 방법
**void return type asynchronous method**  
{% highlight java%}
@Async
public void sampleAsync() {
	//do logic
}
{% endhighlight %}


**return type asynchronous method**
{% highlight java%}
@Async
public Future<String> sampleAsync(){
	//do logic
	return new AsyncResult<String>("Hello world");
}
{% endhighlight %}


**execute method**
{% highlight java%}
@Service
@EnableAsync
public class SampleClass {
	@Autowired
	SampleAsyncClass SampleAsyncClass;

	public void run(){
		Future<String> result = SampleAsyncClass.sampleAsync();
		 while (!(result.isDone()) {
            Thread.sleep(10); 
        }
  
        System.out.println(result.get());
	}
} 
{% endhighlight %}

[Future interface API 문서 바로가기](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html?is-external=true)

---

출처 :
- [spring.io, https://spring.io/guides/gs/async-method/](https://spring.io/guides/gs/async-method/)  
- [http://www.baeldung.com/spring-async](http://www.baeldung.com/spring-async)  
- [Outsider's Dev Story, http://blog.outsider.ne.kr/1066](http://blog.outsider.ne.kr/1066)  

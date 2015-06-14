---
layout: post
title:  "[Java] Arrays.asList"
date:   2015-04-05 00:00:00
categories: posts java
---

## Arrays.asList

---

* 개발을 하다보면 배열을 리스트로 바꾸어야 할 때가 있다.
* java.util에는 Arrays 클래스가 존재한다.
* Arrays.asList는 배열을 리스트로 반환해 준다.

{% highlight java %}
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
{% endhighlight %}

---

* 이 메서드를 이용해서 손 쉽게 배열을 캐스팅하여 컬렉션 관련 유틸들을 이용할 수 있다.
* 하지만 이 메서드에는 몇 가지 제약사항이 있다.  
* JAVA API 문서를 참조하면 해당 메서드에 다음과 같을 설명이 있다.  
`Returns a fixed-size list backed by the specified array.`
* `fixed-size list`를 반환하기 때문에 element를 추가, 삭제가 불가능하다.
* 만약 이를 시도하면 `UnsupportedOperationException`가 발생하게 된다.
* 이를 해결하는 가장 간단한 방법은 새로운 리스트를 만드는 것이다.

{% highlight java %}
List<String> list = new ArrayList(Arrays.asList(array));
{% endhighlight %}

* 위와 같은 방법을 사용하면 새로운 리스트를 만들면 추가, 삭제가 가능하다.

---

- 출처 : [Java7 API Docs, https://docs.oracle.com/javase/7/docs/api/java/util/Arrays.html](https://docs.oracle.com/javase/7/docs/api/java/util/Arrays.html)
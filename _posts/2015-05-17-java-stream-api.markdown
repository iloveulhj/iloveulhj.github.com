---
layout: post
title:  "[JAVA] 스트림 API"
date:   2015-05-17 00:00:00
categories: jekyll update
---

이 포스트는 "가장 빨리 만나는 자바8"의 "2장. 스트림 API"를 학습한 내용으로 이루어져 있습니다. 

---

# 반복에서 스트림 연산으로

**스트림과 컬렉션의 차이**

1. 스트림은 요소들을 보관하지 않는다. 요소들은 하부의 컬렉션에 보관되거나 필요할 때 생성된다.
2. 스트림 연산은 원본을 변경하지 않는다. 대신 결과를 담은 새로운 스트림을 반환한다.
3. 스트림 연산은 가능하면 지연(lazy) 처리된다. 지연 처리란 결과가 필요하기 전에는 실행되지 않음을 의미한다.
예를 들어, 긴 단어를 모두 세는 대신 처음 5개 긴 단어를 요청하면, filter 메서드는 5번 째 일치 후 필터링을 중단한다.
결과적으로 무한 스트림도 만들 수 있다.

**손 쉬운 병렬화**

stream을 parallenStream으로 변경하는 것만으로 스트림 라이브러리가 필터링과 카운팅을 병렬로 수행할 수 있다.
{% highlight java %}
long count = words.stream().filter(w -> w.length() > 12).count();
long count = words.parallelStream().filter(w -> w.length() > 12).count();
{% endhighlight %}

**스트림 API의 3단계**

    orders.스트림 생성().중개 연산().최종 연산() 

1. 스트림을 생성한다.
2. 초기 스트림을 다른 스트림으로 변환하는 중간 연산(intermediate operation)들을 하나 이상의 단계로 지정한다.
3. 결과를 산출하기 위해 최종 연산(terminal operation)을 적용한다. 
이 연산은 앞선 지연 연산(lazy operation)들의 실행을 강제한다. 
이후로는 해당 스트림을 더는 사용할 수 없다.

---

# 참고 자료
출처: ["씹고 뜯고 맛보고 즐기는 스트림 API", YongKwon Park, www.slideshare.net](http://www.slideshare.net/arawnkr/api-42494051)

**스트림 생성자**

| 카테고리 | 메서드 |
| :-- | :-- |
| Collections | streams(), parallelStream() |  
| Arrays | streams(*) |
| Stream ranges | range(...), rangeClosed(...) | 
| Directly from values | of(*) |
| Generators | iterate(...), generate(...) | 
| Resources | lines() |
| Pattern | splitAsStream() | 

**중개 연산자** 

* 스트림을 받아서 스트림을 반환
* 무상태 연산과 내부 유지 연산으로 나누어짐
* 기본적으로 지연(lazy) 연산 처리(성능 최적화)

**최종 연산자**

* 스트림의 요소들을 연산 후 결과 값을 반환
* 최종 연산 시 모든 연산 수행(반복 작업 최소화)
* 이후 더 이상 스트림을 사용할 수 없음

---

# 스트림 생성

컬렉션을 메서드로 변환
{% highlight java %}
Stream<String> stream = Stream.of(contents.split("[\\P{L}]+"));
{% endhighlight %}

of 메서드는 가변 인자 파라미터를 받아 스트림을 생성
{% highlight java %}
Stream<String> stream = Stream.of("Using", "Stream", "API", "From", "Java8");
{% endhighlight %}

배열의 일부에서 스트림을 생성
{% highlight java %}
String[] wordArray = {"Using", "Stream", "API", "From", "Java8"};

// Arrays.stream(array, from, to);
Stream<String> stream = Arrays.stream(wordArray, 0, 4);
{% endhighlight %}

요소가 없는 스트림 생성
{% highlight java %}
Stream<String> stream = Stream.empty();
{% endhighlight %}

무한 스트림을 만드는 generate 정적 메서드(Supplier\<T>)
{% highlight java %}
Stream<String> stream = Stream.generate(() -> "Stream");
Stream<Double> stream = Stream.generate(Math::random);
{% endhighlight %}

무한 스트림을 만드는 iterate 정적 메서드(UnaryOperator\<T>) 
{% highlight java %}
// Seed 값과 함수를 받고, 해당 함수를 이전 결과에 반복적으로 적용
Stream<BigInteger> stream = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
{% endhighlight %}

---

# filter, map, flatMap 메서드

**stream.filter()**
{% highlight java %}
// 특정 조건과 일치하는 모든 요소를 담은 새로운 스트림을 반환
// 필터의 인자는 Predicate<T>, 즉 T를 받고 boolean
Stream<String> longStream = stream.filter(w -> w.length() > 5);
{% endhighlight %}

**stream.map()**
{% highlight java %}
// 스트림에 있는 값들을 특정 방식으로 변환하여 새로운 스트림을 반환
Stream<Character> mapStream = stream.map(s -> s.charAt(0));
{% endhighlight %}

**stream.flatMap()**
{% highlight java %}
// 스트림들을 하나의 스트림으로 합쳐서 하나의 새로운 스트림을 반환
Stream<Character> flatMapStream = stream.flatMap(w -> characterStream(w));

private Stream<Character> characterStream(String s) {
		List<Character> result = new ArrayList<>();
		for (char c : s.toCharArray()) {
			result.add(c);
		}
		return result.stream();
	}
{% endhighlight %}


---

# 서브스트림 추출과 스트림 결합

**stream.limit(n)**
{% highlight java %}
// n개 요소 이후 끝나는 새로운 스트림을 반환
Stream<Double> limitStream = Stream.generate(Math::random).limit(10);
{% endhighlight %}

**stream.skip(n)**
{% highlight java %}
// n개 요소를 버린 후 이어지는 스트림을 반환
Stream<String> skipStream = stream.skip(3);
{% endhighlight %}

**Stream.concat(a, b)**
{% highlight java %}
// 두 스트림을 연결하여 새로운 스트림을 반환
Stream<String> concatStream = Stream.concat(stream1, stream2);
{% endhighlight %}

---

# 상태 유지 변환

앞에서 살펴본 스트림 변환은 무상태 변환이다. 
다시 말해 필터링 또는 매핑된 스트림에서 요소를 추출할 때 결과가 이전 요소에 의존하지 않는다.
몇 가지 상태 유지 변환도 존재한다.

**stream.distinct()**
{% highlight java %}
// 중복 값을 제거한 새로운 스트림을 반환
Stream<String> distinctStream = stream.distinct();
{% endhighlight %}

**stream.sorted()**
{% highlight java %}
// 정렬된 새로운 스트림을 반환
Stream<String> sortedStream = stream.sorted(Comparator.comparing(String::length).reversed());
{% endhighlight %}


---

# 단순 리덕션

리덕션 메서드는 스트림을 프로그램에서 사용할 수 있는 값으로 리듀스한다.
리덕션은 최종 연산이다. 최종 연산을 적용한 후에는 스트림을 사용할 수 없다.
이들 메서드는 전체 스트림을 검사하지만 여전히 병렬 실행(`parallel()`)을 통해 이점을 얻을 수 있다.

**stream.count()**
{% highlight java %}
// 스트림의 요소 갯수를 리턴
long count = stream.count();
{% endhighlight %}

**stream.max()**
{% highlight java %}
// 스트림에서 최대값을 리턴
Optional<String> max = stream.max(String::compareToIgnoreCase);
if (max.isPresent()) {
    System.out.println("max: " + max.get());
}
{% endhighlight %}

**stream.min()**
{% highlight java %}
// 스트림에서 최소값을 리턴
Optional<String> min = stream.min(String::compareToIgnoreCase);
if (min.isPresent()) {
    System.out.println("min: " + min.get());
}
{% endhighlight %}

**stream.findFirst()**
{% highlight java %}
// 스트림에서 비어있지 않은 첫번째 값을 반환
Optional<String> startWithS = stream.filter(s -> s.startsWith("S")).findFirst();
if (startWithS.isPresent()) {
    System.out.println("findFirst: " + startWithS.get());
}
{% endhighlight %}

**stream.findAny()**
{% highlight java %}
// 스트림에서 순서에 상관없이 일치하는 값 하나를 반환
Optional<String> startWithS = stream.filter(s -> s.startsWith("S")).findAny();
if (startWithS.isPresent()) {
    System.out.println("findAny: " + startWithS.get());
}
{% endhighlight %}

**stream.anyMath()**
{% highlight java %}
// 스트림에서 일치하는 요소가 있는지 여부를 반환
boolean aWordStartWithS = stream.anyMatch(s -> s.startsWith("S"));
{% endhighlight %}

---

# 옵션 타입

Optional<T> 객체는 T 타입 객체 또는 객체가 없는 경우의 래퍼다.

**Optional\<T>.get()**   
get 메서드는 감싸고 있는 요소가 존재할 때는 요소를 반환하고 없을 경우는 NoSuchElementException을 던진다. 
{% highlight java %} 
Optional<T> optionalValue = ...;
optionalValue.get().someMethod();
{% endhighlight %}

그러므로 위 예제는 다음 예제보다 안전할 것이 없다. 
{% highlight java %}
T value = ...;
value.someMethod();
{% endhighlight %}

**Optional\<T>.isPresent()**  
isPresent 메서드는 Optional<T> 객체가 값을 포함하는지 알려준다.
{% highlight java %}
if(optionalValue.isPresent()) {
	optionalValue.get().someMethod();
}
{% endhighlight %}

하지만 다음 예제와 크게 달라보이진 않는다.
{% highlight java %}
if(value != null) {
	value.someMethod();
}
{% endhighlight %}

**Optional\<T>.ifPresent()**
{% highlight java %}
// 옵션 값이 존재하면 해당 함수로 전달되며, 그렇지 않으면 아무 일도 일어나지 않음
optionalValue.ifPresent( v -> results.add(v));
optionalValue.ifPresent(results::add);
{% endhighlight %}

**Optional\<T>.map()**
{% highlight java %}
// 값이 존재하면 해당 함수를 호출한 후, Optional<T>를 리턴
// added에는 true, false, null을 가진 Optional을 가질 수 있음
Optional<Boolean> added = optionalValue.map(results::add);
{% endhighlight %}


**Optional\<T>.orElse()**
{% highlight java %}
// 감싸고 있는 문자열, 또는 문자열이 없는 경우는 ""를 리턴
String result = optionalValue.orElse("");
{% endhighlight %}

**Optional\<T>.orElseGet()**
{% highlight java %}
// 감싸고 있는 문자열, 또는 문자열이 없는 경우는 함수를 호출
String result = optionalValue.orElseGet(() -> System.getProperty("user.dir"));
{% endhighlight %}

**Optional\<T>.orElseThrow()**
{% highlight java %}
// 감싸고 있는 문자열, 또는 문자열이 없는 경우는 예외를 발생
String result = optionalValue.orElseThrow(NoSuchElementException::new);
{% endhighlight %}

**Optional\<T>.of(), Optional\<T>.empty()**
{% highlight java %}
public static Optional<Double> inverse(Double x) {
	return x == 0 ? Optional.empty() : Optional.of(1 / x);
}
{% endhighlight %}

**Optional\<T>.ofNullable()**
{% highlight java %}
// obj가 null이면 Optional.empty()를, null이 아니면 Optional.of(obj)를 반환
Optional<String> optionalValue = Optional.ofNullable(obj);
{% endhighlight %}


---

# 리덕션 연산

---

# 결과 모으기

---

# 맵으로 모으기

---

# 그룹핑과 파티셔닝

---

# 기본 타입 스트림

---

# 병렬 스트림

---

# 함수형 인터페이스

대부분의 Stream의 API는 인자로 함수형 인터페이스를 받아서 처리한다.
람다 표현식을 사용한 다음 코드가 있다.
{% highlight java %}
Stream<String> filterStream = stream.filter(s -> s.length() >= 4);
{% endhighlight %}

다음은 위의 예제를 람다 표현식을 이용하지 않고 기존의 스타일로 변경한 코드이다.
{% highlight java %}
Stream<String> filterStream = stream.filter(new Predicate<String>() {
	@Override
	public boolean test(String s) {
		return s.length() >= 4;
	}
});
{% endhighlight %}

* java.util.function.*
* 스트림 API의 대부분은 파라미터로 함수형 인터페이스를 받음
* 함수형 인터페이스는 람다 표현식 또는 메서드 표현식으로 사용

| 함수형 인터페이스 | 파라미터 타입 | 반환 타입 | 설명 |
| :-- | :-: | :-: | :-- |
| Supplier\<T> | 없음 | T | T 타입 값을 공급한다. | 
| Consumer\<T> | T | void | T 타입 값을 소비한다. |
| BiConsumer\<T, U> | T, U | void | T와 U 타입 값을 소비한다. | 
| Predicate\<T> | T | boolean | boolean값을 반환하는 함수다. | 
| ToIntFunction\<T> | T | int | T 타입을 인자로 받고 각각 int 값을 반환하는 함수다. |
| ToLongFunction\<T> | T |long | T 타입을 인자로 받고 각각 long 값을 반환하는 함수다. |
| ToDoubleFunction\<T> | T | double | T 타입을 인자로 받고 각각 double 값을 반환하는 함수다. | 
| IntFunction\<R> | int | R | int를 인자로 받고 R 타입을 반환하는 함수다. |
| LongFunction\<R> | long | R | long을 인자로 받고 R 타입을 반환하는 함수다. |
| DoubleFunction\<R> | double | R | double을 인자로 받고 R 타입을 반환하는 함수다. | 
| Function\<T, R> | T | R | T 타입을 인자로 받고 R 타입을 반환하는 함수다. | 
| BiFunction\<T, U, R> | T, U | R | T와 U 타입을 인자로 받고 R 타입을 반환하는 함수다. | 
| UnaryOperatior\<T> | T | T | T 타입에 적용되는 단항 연산자다. |
| BinaryOperator\<T> | T, T | T | T 타입에 적용되는 이항 연산자다 | 

---




출처: `"가장 빨리 만나는 자바8", 신경근 옮김, 길벗, 2014` 

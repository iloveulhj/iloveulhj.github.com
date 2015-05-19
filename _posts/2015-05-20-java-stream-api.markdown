---
layout: post
title:  "[JAVA] 스트림 API"
date:   2015-05-20 00:00:00
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

**Optional\<T>.flatMap()**  
s.f() = Optional(type T), T.g() = Optional(type U)일 경우 다음과 같은 방법으로 합성할 수 있다.  
{% highlight java %}
Optional<U> result = s.f().flatMap(T::g);
// s.f()는 Optional<T>을 반환하므로 다음과 같은 명령어는 유효하지 않다.
s.f().g();
{% endhighlight %}

{% highlight java %}
@Test
public void flatMap() {
	Optional<Double> result = Optional.of(4.0)
			.flatMap(x -> inverse(x)).flatMap((x -> squareRoot(x)));

	Double actual = result.get();
	Double expected = Math.sqrt(1/4.0);
	
	System.out.println("flatMap: " + actual);
	assertEquals(expected, actual);
}

public Optional<Double> inverse(Double x) {
	return x == 0 ? Optional.empty() : Optional.of(1 / x);
}

public Optional<Double> squareRoot(Double x) {
	return x < 0 ? Optional.empty() : Optional.of(Math.sqrt(x));
}
{% endhighlight %}

---

# 리덕션 연산

**Stream.reduce()**  
스트림의 요소들을 다른 방법으로 결합하고 싶은 경우는 reduce 메서드들 중 하나를 사용하면 된다.
가장 단순한 형태는 이항 함수를 받아서 처음 두 요소부터 시작하여 계쏙해서 해당 함수를 적용한다.

reduce 메서드가 리덕션 연산 op를 가지면, 해당 리덕션은 V0 op V1 op V2 op V3 op ... 를 돌려준다. 
연산 op는 결합 법칙을 지원해야 한다. 즉 결합하는 순서는 문제가 되지 않아야 한다. 
{% highlight java %}
Optional<T> reduce(BinaryOperator<T> accumulator);

Optional<Integer> sum = stream.reduce((x, y) -> x + y);
Optional<Integer> sum = stream.reduce(Integer::sum);
{% endhighlight %}

e op x = x와 같은 항등값이 존재할 때는 첫번째 인자로 항등값을 넣어줄 수 있다.
그러면 반환 값으로 Optional\<T>가 아닌 T를 받을 수 있다.
{% highlight java %}
T reduce(T identity, BinaryOperator<T> accumulator);

int sum = stream.reduce(0, Integer::sum);
{% endhighlight %}

스트림은 병렬화하기 쉽다는 장점이 있다.
값이 누적되는 연산인 경우는 대부분 바로 병렬화를 할 수 없다.
이 경우는 각 부분의 결과를 결합하도록 사용하는 함수를 3번재 인자로 넣어주어야 한다.
{% highlight java %}
<U> U reduce(U identity,
			 BiFunction<U, ? super T, U> accumulator,
			 BinaryOperator<U> combiner);

int result = wordList.parallelStream().reduce(0,
				(Integer total, String word) -> total + word.length(),
				(Integer total1, Integer total2) -> total1 + total2);
{% endhighlight %}

실전에서는 reduce 메서드를 많이 사용하지 않을 것이다.
보통은 숫자 스트림에 매핑한 후에 각 값을 계산해주는 메서드를 이용하는 것이 더 쉽다.
{% highlight java %}
words.mapToInt(String::lnegth).sum();
{% endhighlight %}

---

# 결과 모으기

**배열 만들기**
{% highlight java %}
String[] result = stream.toArray(String[]::new);
{% endhighlight %}

**Stream.collect()**  
병렬화를 지원하면서 한 객체의 스트림의 요소들을 모으려고 할 때 collect 메서드를 사용하게된다.
collect 메서드는 세 가지 인자를 받는다.

* 공급자: 대상 객체의 새로운 인스턴스를 만든다.
* 누산자: 요소를 대상에 추가한다.
* 결합자: 두 객체를 하나로 병합한다.

StringBuilder와 같이 카운트와 합계를 관리하는 객체라면 collect의 대상이 될 수 있다.
{% highlight java %}
HashSet<String> result = stream.collect(HashSet::new, HashSet::add, HashSet::addAll);
{% endhighlight %}

자바에는 세 함수를 제공하는 인터페이스를 가진 Collectors 클래스가 존재한다.
일일이 공급자, 누산자, 결합자를 지정할 필요 없이 간편하게 호출할 수 있다.
{% highlight java %}
List<String> result = stream.collect(Collectors.toList());
Set<String> result = stream.collect(Collectors.toSet());
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));
{% endhighlight %}

결과 값을 하나의 문자열로 모으는 joining메서드도 존재한다.
{% highlight java %}
String result = stream.collect(Collectors.joining());
String result = stream.collect(Collectors.joining(", "));
{% endhighlight %}

**Stream.forEach(), Stream.forEachOrdered()**  
하나 하나의 값에 연산을 하는 방법도 있다.
forEach의 경우에는 병렬스트림에서 순서를 보장할 수 없다. 
스트림 순서대로 조회하고 싶은 경우에는 forEachOrdered를 사용해야 한다.
하지만 이경우는 병렬성이 주는 대부분의 이점을 포기해야 한다.
{% highlight java %}
stream.forEach(System.out::println)l
stream.forEachOrdered(System.out::println)l
{% endhighlight %}

두 메서드 모두 최종 연산으로 스트림을 재사용할 수없다.
만약 재사용을 하고 싶다면 peek 메서드를 사용해야 한다.
{% highlight java %}
Object[] powers = Stream.iterate(1.,0, p -> p * 2)
	.peek(e -> System.out.println("Fetching " + e))
	.limit(20).toArray();
{% endhighlight %}




---

# 맵으로 모으기

**Collections.toMap()**  
맵을 생성해주는 `Collections.toMap()`메서드 같은 경우에는 3개의 인터페이스가 존재한다.
첫번재로 소개할 메서드는 키와 값을 인자로 받는다. `Function.identity()` 현재 인자로 들어온 값을 그대로 반환한다.
{% highlight java %}
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
								Function<? super T, ? extends U> valueMapper) {
	return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
}

Map<String, String> result = stream.collect(Collectors.toMap(Function.identity(), Function.identity()));
{% endhighlight %}

키에 값이 두개 이상이면 컬렉터는 `IllegalStateException`을 던진다.
다음으로 소개할 메서드는 키가 이미 존재할 경우 세번째 인자인 함수를 통해서 값을 재정의 하도록 한다.
{% highlight java %}
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
								Function<? super T, ? extends U> valueMapper,
								BinaryOperator<U> mergeFunction) {
	return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
}

// 기존에 있던 값을 값으로 그대로 사용한다.
Stream<Locale> locales = Stream.of(Locale.getAvailableLocales());
Map<String, String> reesult = locales.collect(
		Collectors.toMap(
				(Locale l) -> l.getDisplayLanguage(),
				(Locale l) -> l.getDisplayLanguage(l),
				(existingValue, newValue) -> existingValue));
{% endhighlight %}

{% highlight java %}
// 키에 복수의 값이 할당되면 HashSet으로 복수 개의 값(HashSet)을 가진 맵을 생성한다.
Stream<Locale> locales = Stream.of(Locale.getAvailableLocales());
		Map<String, Set<String>> result = locales.collect(
				Collectors.toMap(
						(Locale l) -> l.getDisplayCountry(),
						(Locale l) -> Collections.singleton(l.getDisplayLanguage()),
						(Set<String> a, Set<String> b) -> {
							Set<String> r = new HashSet<String>(a);
							r.addAll(b);
							return r;}));
{% endhighlight %}

마지막으로는 세번째 인자로 생성자를 전달하게 되면 해당하는 타입으로 반환하는 메서드이다.
{% highlight java %}
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
							Function<? super T, ? extends U> valueMapper,
							BinaryOperator<U> mergeFunction,
							Supplier<M> mapSupplier) {
	BiConsumer<M, T> accumulator
			= (map, element) -> map.merge(keyMapper.apply(element),
										  valueMapper.apply(element), mergeFunction);
	return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}

Map<String, String> result = stream.collect(Collectors.toMap(
				Function.identity(), 
				Function.identity(),
				(existingValue, newValue) -> { throw new IllegalStateException(); },
				HashMap::new));
{% endhighlight %}



---

# 그룹핑과 파티셔닝

Collectors는 비슷한 성질의 원소들을 분류하는 메서드를 지원한다.

**Collectors.partitionBy()**  
`Collectors.partitionBy()`는 분류함수가 boolean을 반환할 경우 유요항다.
{% highlight java %}
Map<Boolean, List<Locale>> result = locales.collect(
		Collectors.partitioningBy((Locale l) -> l.getLanguage().equals("en")));
{% endhighlight %}

**Collectors.groupingBy()**  
`Collectors.groupingBy()`는 분류함수의 반환값에 따라 그루핑한다.
{% highlight java %}
Map<String, List<Locale>> result = locales.collect(
		Collectors.groupingBy(Locale::getCountry));
{% endhighlight %}

`Collectors.groupingBy()`는 기본적으로 List형태로 그루핑을 하나 다운스트림 컬렉터를 통해 특정 방식으로 처리가 가능하다.
아래 예제 말고도 `Collectors.maxBy()`, `Collectors.mapping()`등의 다운스트림 컬렉터가 존재하다.
{% highlight java %}
Map<String, Set<Locale>> result = locales.collect(
		Collectors.groupingBy(Locale::getCountry, Collectors.toSet()));


Map<String, Long> result = locales.collect(
		Collectors.groupingBy(Locale::getCountry, Collectors.counting()));
{% endhighlight %}

---

# 기본 타입 스트림

스트림 라이브러리는 기본 타입 값들에 특화된 IntStream, LongStream, DoubleStream을 포함한다.
short, char(인코딩의 코드단위로 이용), byte, boolen의 경우는 Intstream을 이용한다.
float인 경우는 DoubleStream을 이용한다. 다음은 기본적인 정적 스트림 생성 예제이다.
{% highlight java %}
IntStream result = IntStream.of(1, 2, 3, 4, 5);
IntStream result = Arrays.stream(array, 0, 5);
{% endhighlight %}

다음은 크기 증가 단위가 1인 정수 범위인 정적 스트림을 생성하는 예제이다.
{% highlight java %}
IntStream result = IntStream.range(0, 5); // 최대값 제외
IntStream result = IntStream.rangeClosed(1, 5); // 최대값 포함
{% endhighlight %}

다음은 객체 스트림을 기본 타입 스트림으로 변환하는 예제이다.
{% highlight java %}
IntStream result = stream.mapToInt(String::length);
{% endhighlight %}

일반적으로 기본 타입 스트림을 대상으로 동작하는 메서드는 객체 스트림 대상 메서드와 유사하다.
다음은 주목할만한 차이점이다.

* toArray 메서드는 기본 타입 배열을 리턴한다.
* OptionalInt, OptionalLong, OptionalDouble을 리턴한다. Optional 클래스와 유사하지만 get 메서드 대신 getAsInt, getAsLong, getAsDouble 등을 포함한다.
* 각각 합계, 평균, 최대값, 최소값을 리턴하는 sum, average, max, min 메서드를 포함한다. 객체 스트림에는 정의되어 있지 않다.
 * summaryStatistic 메서드는 스트림의 합계 , 평균, 최대값, 최소값ㅇ르 동시에 보고할 수 있는 IntSummaryStatistics, LongSummaryStatistics, DoubleSummaryStatistics 타입을 반환한다. 

---

# 병렬 스트림

`Stream.sorted()`를 호출해서 얻는 스트림은 순서를 유지한다.
순서 유지 스트림의 결과들은 원본 요소들의 순서대로 쌓이고, 전체적으로 예측 가능하게 동작한다.

`Stream.unordered()`를 호출해서 얻는 스트림은 순서에 상관 없을을 나타낸다.
'순서'를 포기함으로서 `Stream.distinct()`, `Stream.limit()`등과 같은 스트림들은 더 좋은 성능을 낼 수 있다.

`Collectors.groupingByConcurrent()` 메서드는 공유되는 병행맵을 사용한다.
병렬화의 이점을 얻기 위해 맵 값들의 순서는 스트림 순서와 달라진다. 
이 컬렉터는 심지어 순서 유지 스트림에서도 순서가 달라진다. 
그럼에도 이 스트림은 병렬로 만들어서 사용해야 한다.

스트림 연산을 수행하는 동안에는 컬렉션을 수정하면 안된다.
스트림은 자체적으로 데이터를 모으지 않음을 명심한다. 
해당 컬렉션을 수정하면 스트림 연산들의 결과는 정의되지 않는다.
이 경우는 순차 스트림, 병렬 스트림 모두 해당한다.
정확히는 최종 연산이 실행되는 시점에 컬렉션이 변경되면 안된다.
{% highlight java %}
List<String> wordList = new ArrayList<>(Arrays.asList(new String[]{"HELLO", "WORLD", "Java"}));
Stream<String> words = wordList.stream();
wordList.add("END");
long n = words.distinct().count();
{% endhighlight %}


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

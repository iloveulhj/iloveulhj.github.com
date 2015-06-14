---
layout: post
title:  "[Java] Collections.sort"
date:   2014-09-30 00:00:00
categories: posts java
---

#  정렬 알고리즘의 선택
---
**Collections.sort(List), Collection.sort(List, Comparator), Arrays.sort(Object\[\])**

> JDK6 - Arrays.mergeSort 메서드를 이용  
> JDK7,8 - Arrays.legacyMergeSort와 ComparableTimSort.sort 메서드를 이용  

**Arrays.sort(double\[\]), Arrays.sort(int\[\]), Arrays.sort(char\[\]), Arrays.sort(long\[\]), Arrays.sort(float\[\]), Arrays.sort(byte\[\])**

> JDK6 - Arrays.sort1과 Arrays.sort2(실수형 자료) 메서드를 이용  
> JDK7,8 - DualPivotQuicksort.sort 메서드를 이용  

--- 

# 코드 분석
---
**Collections.sort(List), Collection.sort(List, Comparator), Arrays.sort(Object\[\])**  

OpenJDK 6-b27  
{% highlight java %}
//Collections.sort(List<T>)
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    Object[] a = list.toArray();
    Arrays.sort(a);
    ListIterator<T> i = list.listIterator();
    for (int j=0; j<a.length; j++) {
        i.next();
        i.set((T)a[j]);
    }
}
{% endhighlight %}
> 배열로 변환한 후 Arrays.sort(Object\[\])로 정렬을 위임합니다.

{% highlight java %}
//Arrays.sort(Object[])
public static void sort(Object[] a) {
    Object[] aux = (Object[])a.clone();
    mergeSort(aux, a, 0, a.length, 0);
}
{% endhighlight %}
> 즉 JDK6에서는 Collections.sort는 Arrays.mergeSort 메서드를 이용하여 정렬을 합니다.  
  
  
OpenJDK 7u40-b43
{% highlight java %}
//Collections.sort(List<T>)
public static <T extends Comparable<? super T>> void sort(List<T> list) {
    Object[] a = list.toArray();
    Arrays.sort(a);
    ListIterator<T> i = list.listIterator();
    for (int j=0; j<a.length; j++) {
        i.next();
        i.set((T)a[j]);
    }
}
{% endhighlight %}
> 정렬 알고리즘은 Arrays.sort(Object\[\])로 위임합니다.  
  
{% highlight java %}
//Arrays.sort(Object[])
public static void sort(Object[] a) {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a);
        else
            ComparableTimSort.sort(a);
}
{% endhighlight %}
> LegacyMergeSort.userRequested를 통해 legacyMergeSort(Object\[\])와 ComparableTimSort.sort(Object\[\])중 하나를 사용합니다.  
  
{% highlight java %}
/--
 - Old merge sort implementation can be selected (for
 - compatibility with broken comparators) using a system property.
 - Cannot be a static boolean in the enclosing class due to
 - circular dependencies. To be removed in a future release.
 -/
static final class LegacyMergeSort {
    private static final boolean userRequested =
        java.security.AccessController.doPrivileged(
            new sun.security.action.GetBooleanAction(
                "java.util.Arrays.useLegacyMergeSort")).booleanValue();
}
{% endhighlight %}
> 주석에는 broken comparators(?)의 호환성을 위해 merge sort 구현체가 사용될 수 있다고 합니다.  
> 그 이외의 경우는 Tim sort를 이용하여 정렬합니다.  
> 즉 JDK7이상의 버전에서는 Collections.sort는 Arrays.sort를 호출하고 결국은 ComparableTimSort.sort 메서드를 통해 정렬합니다.  

---
**Arrays.sort(double\[\]), Arrays.sort(int\[\]), Arrays.sort(char\[\]), Arrays.sort(long\[\]), Arrays.sort(float\[\]), Arrays.sort(byte\[\])**    
  
OpenJDK 6-b27
{% highlight java %}
//Arrays.sort(int\[\]), Arrays.sort(char\[\]), Arrays.sort(long\[\]), Arrays(byte\[\])
public static void sort(int[] a) {
    sort1(a, 0, a.length);
}
...
{% endhighlight %}
> sort1 메서드를 통해 정렬합니다.  

{% highlight java %}
//Arrays.sort(float\[\]), Arrays.sort(double\[\])
public static void sort(double[] a) {
    sort2(a, 0, a.length);
}
...
{% endhighlight %}
> 소수점이 들어갔을 경우는 sort2 메서드를 통해 정렬합니다.  
> 즉 실수일 경우는 Arrays.sort2 메서드를 통해 그 외는 Arrays.sort1 메서드를 통해 정렬합니다.    
  
OpenJDK 7u40-b43  
  
{% highlight java %}
//Arrays.sort(double\[\]), Arrays.sort(int\[\]), Arrays.sort(char\[\]), Arrays.sort(long\[\]), Arrays.sort(float\[\]), Arrays.sort(byte\[\])
public static void sort(double[] a) {
    DualPivotQuicksort.sort(a);
}
...
{% endhighlight %}
> JDK7이상의 버전에서는 DualPivotQuicksort.sort 메서드를 통해서 정렬합니다. 

---
**코드 분석(JDK 6) - Arrays.mergeSort()**  
  
> Insertion sort와 Merge sort를 결합한 정렬 알고리즘을 사용합니다.  

{% highlight java %}
//길이가 6이 넘는 List가 들어왔을 때 쪼개지는 가장 큰 단위는 6입니다.
//6이하의 정렬은 Insertion sort를 사용합니다.
private static final int INSERTIONSORT_THRESHOLD = 7;

void mergeSort(Object[] src, Object[]dest, ...) {
     if(길이 < INSERTIONSORT_THRESHOLD) {
          Insertion sort(dest);
          return;
     }

     반으로 분할하기;
     mergeSort(앞부분);
     mergeSort(뒷부분);

     if(정렬이 되어있는 경우) {
          dest에 복사하기;
          return;
     }

     for(dest 전체 순회) {
          비교후 정렬;
     }
}
{% endhighlight %}

---
**코드 분석(JDK 7,8) - ComparableTimSort.sort(Object\[\])**  

> 사용된 정렬 알고리즘  
> - Binary Insertion Sort - MIN_MERGE 이하  
> - Tim sort - MIN_MERGE 이상  
{% highlight java %}
유효성 검사
//Binary Insertion sort
if( 정렬할 길이 < MIN_MERGE ) { // MIN_MERGE의 기본값 = 32
     return Binary Insertion Sort로 정렬 // 기존의 Insertion sort에 삽입할 위치를 탐색하는 알고리즘으로 Binary search를 사용한다.
}
//Tim sort
ComparableTimSort 객체 생성
do{
     run 생성
     if( run의 길이 < minRun ) { //minRun의 크기는 정렬할 길이에 따라 변경
           Binary Insertion Sort를 사용하여 run의 길이를 강제로 늘림
     }
     ComparableTimSort.push() // run의 정보를 stack에 푸쉬
     ComparableTimSort.mergeCollapse() // run단위로 병합과 정렬
}while(List의 끝까지 순회하였다)
ComparableTimSort.mergeForceCoolapse() // 1개의 run으로 병합
{% endhighlight %}

---
**Tim sort**  
  
run 생성  
![](/images/collection-sort/collection-sort1.png)

> A natural run : List안에 존재하는 정렬된 sub-array입니다.  
> 정방향, 역방향으로 연속된 run을 찾습니다. 역방향일 경우는 정방향으로 변환하여줍니다.  
  
  
push  

> run의 시작점을 runBase라는 스택에 저장합니다.  
> run의 길이를 runLen이라는 스택에 저장합니다.  
    
mergeCollapse  

![](/images/collection-sort/collection-sort2.png)  
  
> run의 length가 minRun보다 작은 경우, Binary insertion sort를 이용해 길이를 늘려줍니다.  
> length\[top\] + length\[top-1\] < length\[top-2\]  
> length\[top\] < length\[top-1\]  
> 위 두 식을 만족할때까지 병합을 합니다.  
> 이 부분 때문에 O(n log n)을 보장(?)한다고 합니다.  

![](/images/collection-sort/collection-sort3.png)  

> 이웃하는 두 개의 run을 병합하니다.  
> 두 run은 모두 정렬되어 있는 상태이기 때문에  

> > 앞에 있는 run을 복사해서 앞 run의 마지막 값보다 작아지는 뒷 run의 인덱스 다음에 복사하면 뒷 run의 요소 중에 앞 run의 끝보다 큰 부분은 정렬을 생략할 수 있습니다.  
> > 마찬가지로 뒷 run을 복사해서 뒷 run의 처음 값보다 커지는 앞 run의 인덱스 전에 복사하면 앞 run의 요소 중에 뒷 run의 앞보다 작은 부분은 정렬을 생략할 수 있습니다.

> 위에 삽입할 위치를 검색해서 생략을 하는 과정을 gallop mode라 합니다.
> 이후는 두 값을 비교하며 정렬합니다.

![](/images/collection-sort/collection-sort4.png)  

> 만약 남은 두 run중에 한 곳에서 연속해서 삽입할 값이 선택되는 경우는 다시 위에서 진행한 gallop mode를 통해 정렬을 생략하는 시도를 합니다.

결론  

> 랜덤 데이터보다는 정렬이 되있는 데이터들에 정렬을 생략할 수 있기 때문에 더 좋은 성능을 낼 수 있습니다.  
> Worst case도 O(n log n)를 보장합니다.  
> Merge sort + Insertion Sort  

이미지 출처 : [http://en.wikipedia.org/wiki/Timsort](http://en.wikipedia.org/wiki/Timsort)

---
**코드 분석(JDK 6) - Arrays.sort1(), Arrays.sort2()**  

> JDK 6에서 Array.sort()를 호출할 경우  
> Arrays.sort1(), Arrays.sort2() 두 종류의 sort함수가 내부적으로 수행된다.  

Arrays.sort1()

> 정렬 알고리즘이 배열의 크기에 따라 나뉘는데 배열의 크기가 7개 이하일 경우 insertion sort를 호출.  
> 그 외의 경우 quick sort  

Arrays.sort2()의 경우 float, double과 같이 실수(부동소수)data type을 정렬할 때 호출된다.

> sort2(float\[\] a)  
> sort2(double\[\] a)  

정렬과정에서 \-0.0과 NaN 비교하는 불필요한 연산을 피하기 위해 이를 0.0(float은 0.0f, double은 0.0D)으로 전환해주기 위해 내부적으로 int i = Float.floatToIntBits(-0.0F)이나 long l = Double.doubleToLongBits(-0.0D) 를 호출한다.
배열의 \-0.0, NaN을 수정후 sort1()을 호출한다.

---
**코드 분석(JDK 7,8) - DualPivotQuicksort.sort(int\[\], int, int)**  

JDK 1.8 Arrays.sort() 에서는 내부적으로 Dual-Pivot QuickSort 을 사용한다. O( n log( n )) 효율.
{% highlight java %}
Static static void Arrays.sort(int[] a) {
  DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}
{% endhighlight %}
> DualPivotQuicksort에서도 무조건 DualPivotQuicksort를 수행하는것이 아니라  
> 정렬해야할 대상의 크기에 따라 insertion/merge 정렬을 수행할 수 있다.

![](/images/collection-sort/collection-sort5.png)

> 우선 일반적인 quick 정렬이 Pivot이 하나를 사용한다면  
> DualPivotQuicksort은 이름에서 알 수 있듯 Pivot을 두 개 사용한다.  
> 수도코드는 다음과 같다.

{% highlight java %}
1. 배열의 크기가 특정 숫자보다 작을경우 insertion 정렬을 수행한다.  
(적정 배열사이즈와 insertion 정렬을 수행하는 이유는 조사중)

2. 두 pivot 요소 p1, p2를 배열에서 고른다.  
예를들어 배열을 첫 요소를 p1, 배열의 마지막 요소를 p2로 지정한다.

3. p1은 p2보다 반드시 작아야하며 커질경우 서로 swap된다.  
그리고 다음과 같은 part들로 구성한다.  
    part1: left+1 에서 L-1 까지 p1보다 작은 값을 가지는 요소를 가진다.
    part2: L에서 K-1까지 p1보다 크거나 같고 p2보다 작거나 같은 요소를 가진다.
    part3: G+1에서 right-1 까지 p2보다 큰 값을 가지는 요소를 가진다.
    part4: K부터 G까지 나머지 요소들을 가진다.

4. part4의 a[K]의 다음 요소가 p1, p2와 비교하여 이에 대응되는 part1, part2, part3 로 이동된다.

5. 포인터 L, K, G가 바뀐다.

6. 4,5번 과정이 K <= G 일 동안 반복된다.

7. 피봇 p1이 part1의 마지막 요소와 swap된다.  
피봇 p2는 part3의 첫번째 요소와 swap된다.

The steps 1 - 7 are repeated recursively for every part I, part II, and part III.  
8. 이 1~7과정들이 각 part1, part2, part3에서 재귀적으로 일어난다.  
{% endhighlight %}

자세한 절차는 아래 링크에서 확인할 수 있다.  
[`http://www.slideshare.net/sebawild/average-case-analysis-of-java-7s-dual-pivot-quicksort`](http://www.slideshare.net/sebawild/average-case-analysis-of-java-7s-dual-pivot-quicksort)

---
# 성능 분석  

**시간 복잡도**  

| | Timsort | Introsort | Merge sort | Quicksort | Insertion sort | Selection sort | Smoothsort |
| Best case | O\(n) | | O(n log n) \\ | O(n log n) | O\(n) | O(n^2) | O\(n) |
| Average case | O(n log n) | O(n log n) | O(n log n) | O(n log n) | O(n^2) | O(n^2) | O(n log n) |
| Worst case | O(n log n) | O(n log n) | O(n log n) | O(n^2) | O(n^2) | O(n^2) | O(n log n) |

**공간 복잡도**   

| | Timsort | Merge sort | Quicksort | Insertion sort | Selection sort | Smoothsort |
| Space complexity | O\(n) | O\(n) | O(log n) | O(1) | O(1) | O(1) |

출처 : 위키피디아

---

**참고 사이트**  
[`OpenJDK-7u40-b43(Grepcode)`](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/java/util/Collections.java#Collections.sort%28java.util.List%2Cjava.util.Comparator%29)  
[`Timsort(Wikipedia)`](http://en.wikipedia.org/wiki/Timsort)  
[`MergeSort(VISUALGO)`](http://www.comp.nus.edu.sg/~stevenha/visualization/index.html)  
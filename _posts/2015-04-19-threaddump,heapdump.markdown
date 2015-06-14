---
layout: post
title:  "[Java] Threaddump, Heapdump"
date:   2015-04-19 00:00:00
categories: posts java
---

# Head dump, Thread dump

2015년 4월 17일에 사내에서 진행한 자바 관련 교육을 참가하였다. 자바 트러블 슈팅에 대해 '자바의신'의 작가이신 이상민 엔지니어님께서 진행해주셨다.
개발이 아닌 운영이라는 측면의 자바에 대해 알아 볼 수 있는 시간이었다. 사내 교육에 나왔던 분석 방법들 중 쓰레드 덤프와 힙 덤프에 대해 간단하게 알아보도록 하자.

---

# 트러블슈팅

자바 어플리케이션에서는 다양한 종료의 문제가 발생할 수 있다.

* 전반적으로 느려지는 현상  
* 헹이 걸리는 현상  
* 크래쉬  
* etc...

동일한 증상이라도 다양한 원인이 존재한다.  
트러블 슈팅을 할 때는 근거자료를 기반으로 모든 가능성을 염두에 두고 분석하도록 하자.  

* 쓰레드 덤프: 시스템이 느리거나, 헹이 걸릴 경우  
* 힙 덤프: 크래시(OOM), 메모리 릭의 경우
* fatal log: 크래시

---

# 쓰레드 덤프(Ctrl-Break Handler)
  
`kill -QUIT pid`, `kill -3 pid`  
명령어를 이용해 해당 application의 standard out을 통해 출력  

**쓰레드 덤프**  
쓰레드 덤프는 JVM 위의 모든 쓰레드에 대해 쓰레드의 상태와 쓰레드 스택을 포함한다.  
쓰레드 덤프는 어플리케이션을 종료하지 않고, 어플리케이션의 현재 쓰레드 상태를 출력한다.  
쓰레드 덤프에는 Thread dump,  deadlock detection, heap summary이 포함된다.  

**Thread stack**  
{% highlight java%}
Full thread dump Java HotSpot(TM) Client VM (1.6.0-rc-b100 mixed mode):

"DestroyJavaVM" prio=10 tid=0x00030400 nid=0x2 waiting on condition [0x00000000..0xfe77fbf0]
   java.lang.Thread.State: RUNNABLE

"Thread2" prio=10 tid=0x000d7c00 nid=0xb waiting for monitor entry [0xf36ff000..0xf36ff8c0]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at Deadlock$DeadlockMakerThread.run(Deadlock.java:32)
        - waiting to lock <0xf819a938> (a java.lang.String)
        - locked <0xf819a970> (a java.lang.String)

"Thread1" prio=10 tid=0x000d6c00 nid=0xa waiting for monitor entry [0xf37ff000..0xf37ffbc0]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at Deadlock$DeadlockMakerThread.run(Deadlock.java:32)
        - waiting to lock <0xf819a970> (a java.lang.String)
        - locked <0xf819a938> (a java.lang.String)

"Low Memory Detector" daemon prio=10 tid=0x000c7800 nid=0x8 runnable [0x00000000..0x00000000]
   java.lang.Thread.State: RUNNABLE

"CompilerThread0" daemon prio=10 tid=0x000c5400 nid=0x7 waiting on condition [0x00000000..0x00000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" daemon prio=10 tid=0x000c4400 nid=0x6 waiting on condition [0x00000000..0x00000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" daemon prio=10 tid=0x000b2800 nid=0x5 in Object.wait() [0xf3f7f000..0xf3f7f9c0]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0xf4000b40> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:116)
        - locked <0xf4000b40> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:132)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:159)

"Reference Handler" daemon prio=10 tid=0x000ae000 nid=0x4 in Object.wait() [0xfe57f000..0xfe57f940]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0xf4000a40> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:485)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:116)
        - locked <0xf4000a40> (a java.lang.ref.Reference$Lock)

"VM Thread" prio=10 tid=0x000ab000 nid=0x3 runnable 

"VM Periodic Task Thread" prio=10 tid=0x000c8c00 nid=0x9 waiting on condition 
{% endhighlight %}
> 출처: [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps)
  
thread name, thread priority, thread id, native thread id, thread state, address range 등이 포함된다.

**Thread.State**  
`NEW` 시작되지 않은 쓰레드  
`RUNNABLE` jvm에서 실행중인 쓰레드  
`BLOCKED` monitor lock을 기다리며 block된 쓰레드  
`WAITING` 특정 액션을 수행하기 위해 무기한으로 대기중인 쓰레드(idle 상태의 쓰레드)  
`TIMED_WAITING` idle 상태의 쓰레드로 특정 시간이 지나면 사라지는 쓰레드  
`TERMINATED` 종료된 쓰레드  

**Deadlock Detection**  
{% highlight java%}
Found one Java-level deadlock:
=============================
"Thread2":
  waiting to lock monitor 0x000af330 (object 0xf819a938, a java.lang.String),
  which is held by "Thread1"
"Thread1":
  waiting to lock monitor 0x000af398 (object 0xf819a970, a java.lang.String),
  which is held by "Thread2"

Java stack information for the threads listed above:
===================================================
"Thread2":
        at Deadlock$DeadlockMakerThread.run(Deadlock.java:32)
        - waiting to lock <0xf819a938> (a java.lang.String)
        - locked <0xf819a970> (a java.lang.String)
"Thread1":
        at Deadlock$DeadlockMakerThread.run(Deadlock.java:32)
        - waiting to lock <0xf819a970> (a java.lang.String)
        - locked <0xf819a938> (a java.lang.String)

Found 1 deadlock.
{% endhighlight %}  
> 출처: [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps)

JVM에서 Thread에 deadlock이 걸려있는지 쉽게 알 수 있다. 하지만 이 툴에 안잡히는 경우도 (starvation 등) 많으므로 쓰레드덤프 분석도구를 이용하여 분석하는 것이 좋다.


**Heap summary**  
{% highlight java%}
Heap 
  def new generation   total 1152K, used 435K [0x22960000, 0x22a90000, 0x22e40000)
  eden space 1088K,  40% used [0x22960000, 0x229ccd40, 0x22a70000)
  from space 64K,   0% used [0x22a70000, 0x22a70000, 0x22a80000)
  to   space 64K,   0% used [0x22a80000, 0x22a80000, 0x22a90000)
 tenured generation   total 13728K, used 6971K [0x22e40000, 0x23ba8000, 0x26960000)
   the space 13728K,  50% used [0x22e40000, 0x2350ecb0, 0x2350ee00, 0x23ba8000)
 compacting perm gen  total 12288K, used 1417K [0x26960000, 0x27560000, 0x2a960000)
   the space 12288K,  11% used [0x26960000, 0x26ac24f8, 0x26ac2600, 0x27560000)
    ro space 8192K,  62% used [0x2a960000, 0x2ae5ba98, 0x2ae5bc00, 0x2b160000)
    rw space 12288K,  52% used [0x2b160000, 0x2b79e410, 0x2b79e600, 0x2bd60000)
{% endhighlight %}  
> 출처: [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps)
  
Heap summary를 일정 주기 간격으로 출력하여 확인하면 메모리 변화량을 모니터링할 수 있다. GC가 정상적으로 수행되어지는지, 메모리 튜닝이 필요한지 등을 알 수 있다. 

---

# 힙 덤프(HPROF)

힙 덤프는 The Heap Profiler (HPROF) tool을 이용하여 작성할 수 있다.  
힙 덤프 출력 방법은 다양하게 있으니 찾아보기를 바란다.

{% highlight java%}
HEAP DUMP BEGIN (39793 objects, 2628264 bytes) Wed Oct 4 13:54:03 2006
ROOT 50000114 (kind=<thread>, id=200002, trace=300000)
ROOT 50000006 (kind=<JNI global ref>, id=8, trace=300000)
ROOT 50008c6f (kind=<Java stack>, thread=200000, frame=5)
:
CLS 50000006 (name=java.lang.annotation.Annotation, trace=300000)
    loader        90000001
OBJ 50000114 (sz=96, trace=300001, class=java.lang.Thread@50000106)
    name        50000116
    group        50008c6c
    contextClassLoader    50008c53
    inheritedAccessControlContext    50008c79
    blockerLock    50000115
OBJ 50008c6c (sz=48, trace=300000, class=java.lang.ThreadGroup@50000068)
    name        50008c7d
    threads    50008c7c
    groups        50008c7b
ARR 50008c6f (sz=16, trace=300000, nelems=1, 
     elem type=java.lang.String[]@5000008e)
    [0]        500007a5
CLS 5000008e (name=java.lang.String[], trace=300000)
    super        50000012
    loader        90000001
:
HEAP DUMP END
{% endhighlight %}  
> 출처: [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gblvj](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gblvj)

힙 덤프는 format option을 통해 ASCII나 binary format으로 작성할 수 있다.(jhat 유틸리티를 참고하기)  
힙 덤프는 매우 큰 파일이며 힙에 있는 모든 오브젝트들에 대한 기록을 가지고 있다.  
이 파일을 통해 힙 메모리에 대해 확인이 가능하며 메모리 릭과 같은 현상을 파악할 수 있다.  
분석도구를 사용하여 분석하는 것을 권장한다.

---

출처 :  
- [Troubleshooting guide for hotspot VM](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/toc.html)  
- [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gbmps)  
- [http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gblvj](http://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/tooldescr.html#gblvj)
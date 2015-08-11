---
layout: post
title:  "[Java]  JLS Execution"
date:   2015-02-15 00:00:00
categories: jekyll update
---

이 포스트는 [`The Java® Language Specification, 12.Execution`](http://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.1)을 정리한 내용입니다.

---

#Loading of Classes and Interfaces

로딩은 바이너리 형태의 클래스나 인터페이스를 수시로 계산되어지는 특정한 이름을 통해 찾는 과정을 의미하지만 더 일반적으로는 이전에 소스코드에서 자바 컴파일러로부터 계산되고 만들어진 이진 형태의 클래스 오브젝트(클래스, 인터페이스를 나타내는 오브젝트)를 찾는 것을 뜻한다.

이진 포맷의 클래스나 인터페이스는 일반적으로 JVM 명세에 묘사된 .class의 형태이지만 명세를 만족시킨다면 다른 포맷도 가능하다. ClassLoader의 defineClass메소드는 클래스 파일의 이진 표기로부터 클래스 오브젝트를 생성할 것이다.

잘 만들어진 클래스로더들은 이 요소들을 지킨다.  

* 같은 이름이 주어지면, 좋은 클래스로더는 항상 같은 클래스 오브젝트를 반환한다.
* 클래스로더 L1이 C 클래스의 로딩을 클래스로더 L2에 위임하면, 타입 T가 C의 상위클래스, 상위인터페이스, C의 필드, C의 메소드의 파라미터, C생성자, C의 메소드안에 리턴 타입일 때 같은 클래스오브젝트를 반환해야 한다.

악의적인 클래스로더는 위 요소들을 위반할 수 있다. JVM에서 막아주기 때문에 클래스로더는 타입시스템의 보안을 약화시킬 수 없다

---

***The Loading Process***
 
로딩 프로세스는 ClassLoader클래스, 서브클래스로 구현되어진다.  
ClassLoader의 서브클래스들은 다른 로딩정책을 구현할 수 있다. 특별하게 어떤 클래스로더는 클래스와 인터페이스의 바이너리 리프리젠테이션을 캐시, 예상되는 곳에 프리패치, 관련있는 클래스 그룹을 로드할 수 있다. 이러한 활동은 동작중인 어플리케이션의 완벽한 투명하지 않을 수 있다. 예를 들어 클래스로더는 오래된 버전의 클래스를 캐시하고 있어 새로 컴파일된 버전의 클래스를 찾지 못할 수 있다. 프로그램안에서 프래패치와 그룹 로딩이 없을 때 발생할 수 있는 로딩에러를 나타내는 것은 클래스로더의 몫이다.

만약 클래스 로딩에 에러가 발생하면 `ClassCircualrityError`, `ClassFormatError`, `NoClassDefFoundError`와 그 서브클래스 에러들이 발생할 수 있다.

---

#Linking of Classes and Interfaces

#Initialization of Classes and Interfaces

#Creation of New Class Instances

#Finalization Of Class Instances

#Unloading of Classes and Interfaces

#Program Exit
--- 

출처: [`The Java® Language Specification, Chapter 12. Execution`](http://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.1) 
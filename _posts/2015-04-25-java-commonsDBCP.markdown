---
layout: post
title:  "[Java] Commons DBCP 이해하기"
date:   2015-04-25 00:00:00
categories: jekyll update
---

이 포스트는 "네이버를 만든 기술, 읽으면서 배운다 - 자바편"의 "17.Commons DBCP 이해하기"를 정리한 내용입니다. 

---

# 커넥션 풀 라이브러리

데이터베이스와 애플리케이션을 효율적으로 연결하기 위한 필수요소이다. 
커넥션 풀 라이브러리를 잘 사용하면 데이터베이스와 애플리케이션의 일부에서 발생하는 문제를 전체로 전파되지 않게 할 수 있고, 일시적인 문제가 긴 시간 이어지지 않게 할 수 있다. 
웹 애플리케이션의 요청은 대부분 DBMS로 연결되기 때문에 커넥션 풀 라이브러리의 설정은 전체 애플리케이션의 성능과 안정성에 영향을 미치는 핵심이다.

---

# JDK 버전과 Commons DBCP 버전

JDK 버전에 따라 JDBC의 인터페이스가 조금씩 다르므로 JDK버전에 맞는 Commons DBCP 버전을 선택해야 안정된 작동을 기대할 수 있다.

> Commons DBCP 버전에 대응하는 JDK 버전과 JDBC 버전

| Commons DBCP 버전 | JDK 버전 | JDBC 버전 |
| Commons DBCP 2 | JDK 7 | JDBC 4.1 |
| Commons DBCP 1.4 | JDK 6 | JDBC 4 |
| Commons DBCP 1.3 | JDK 1.4 ~1.5 | JDBC 3 |

> Commons DBCP 2에서 이름이 바뀐 Commons DBCP1의 속성

| Commons DBCP 1 | Commons DBCP 2 |
| maxActive | maxTotal |
| maxWait | maxWaitMills |
| removeAbandoned | removeAbandonedOnBorrow / removeAbandonedOnMaintenance | 

Commons DBCP 2는 하위 호환성을 보장하지 않는다.

---

# 커넥션의 검사와 정리

유효성 검사 쿼리(validation query)와 Evictor 스레드 관련 설정으로도 애플리케이션의 안정성을 높일 수 있다.

---

**유효성 검사 쿼리**

`testOnBorrow`: 커넥션 풀에서 커넥션을 얻어올 때 테스트 실행(기본값: true)  
`testOnReturn`: 커넥션 풀로 커넥션을 반환할 때 테스트 실행(기본값: false)  
`testWhileIdle`: Evictor 스레드가 실행될 때 (timeBetweenEvictionRunMillis > 0) 커넥션 풀 안에 있는 유휴 상태의 커넥션을 대상으로 테스트 실행(기본값: false)

ValidationQuery 옵션에는 다음과 같이 쿼리를 설정하기를 권장한다.  

오라클: select 1 from dual  
마이크로소프트 SQL 서버: select 1  
MySQL: select 1  
큐브리드: select 1 from db_root  

`testOnBrrow`, `testOnReturn`은 false, `testWhileIdle`은 true로 설정하기를 권장한다. 
큐브리드의 경우는 자체적으로 커넥션을 관리하고 자동으로 다시 연결하도록 구현되어 있어 `testWhileIdle` 옵션을 false로 설정하기를 권장한다.
 
 
---

**Evictor 스레드와 관련된 속성**

Evictor 스레드는 Commons DBCp 내부에서 커넥션 자원을 정리하는 구성 요소이며 별도의 스레드로 실행된다.

`timeBetweenEvictionRunsMillis`: Evictor 스레드가 작동하는 간격. 기본값은 -1이며 Evictor 스레드의 실행이 비활성화돼 있다.  
`numTestPerEvictionRun`: Evictor 스레드 작동 시 한 번에 검사할 커넥션의 갯수  
`minEvictableIdleTimeMillis`: Evictor 스레드 작동 시 커넥션의 유휴 시간을 확인해 설정값 이상일 경우 커넥션을 제거한다(기본값: 30분)  

---

# statement pooling 관련 옵션

statement pooling 은 JDBC 3.0에 정의된 명세이다. 
JDBC 드라이버가 3.0 명세를 지원하지 않으면 사용할 수 없는 기능이다.
JDBC 2.0 명세만 지원하는 JDBC 드라이버를 사용할 때도 커넥션 풀로 Commons DBCP를 사용하고 있다면 `poolPreparedStatements` 옵션을 true로 설정해 statement pool로도 사용할 수 있다. 
이 때는 반드시 `maxOpenPreparedStatements`옵션을 같이 사용해야 한다. 
`maxOpenPreparedStatements` 옵션은 커넥션마다 PreparedStatement가 할당된다.  
전체 풀링한 PreparedStatement = `connections 수 * maxOpenPreparedStatements` 이다.
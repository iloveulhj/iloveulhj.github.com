---
layout: post
title:  "[SQL] SQL 활용 #2"
date:   2015-06-07 00:00:00
categories: jekyll update
---

이 포스트는 "SQL 전문가 가이드"의 "2-2장. SQL 활용"을 학습한 내용으로 이루어져 있습니다. 

---

# 그룹 함수(GROUP FUNCTION)

#### 데이터 분석 개요

ANSI/ISO SQL 표준은 데이터 분석을 위해서 다음 3가지 함수를 정의하고 있다.

* AGGREGATE FUNCTION
    * COUNT, SUM, AVG, MAX, MIN ... 
* GROUP FUNCTION
    * ROLLUP, GROUP BY, CUBE, GROUPING SETGS
* WINDOW FUNCTION
    * 분석 함수(ANALYTIC FUNCTION), 순위 함수(RANK FUNCTION)

#### ROLLUP 함수
{% highlight sql %}
SELECT DNAME, GROUPING(DNAME),
    JOB,   GROUPING(JOB),
    COUNT(*) "Total Empl",
    SUM(SAL) "Total Sal"
FROM   EMP, DEPT
WHERE  DEPT.DEPTNO = EMP.DEPTNO
GROUP BY ROLLUP (DNAME, JOB)
ORDER BY DNAME, JOB;
{% endhighlight %}

* L1 - GROUP BY 수행시 생성되는 표준 집계
* L2 - DNAME 별 모든 JOB의 SUBTOTAL
* L3 - GRAND TOTAL

ROLLUP, CUBE, CGROUPING SETS 등 새로운 글부 함수를 지원하기 위해 GROUPING 함수가 추가되었다.

* ROLLUP이나 CUBE에 의한 소계가 계산된 결과에는 GORUPING(EXPR) = 1이 표시되고,
* 그 외의 결과에는 GROUPING(EXPR) = 0이 표시된다.

#### CUBE 함수
{% highlight sql %}
SELECT 
    CASE GROUPING(DNAME) WHEN 1 THEN 'ALL DEPARTMENTS' ELSE DNAME END AS DNAME,
    CASE GROUPING(JOB) WHEN 1 THEN 'ALL JOBS' ELSE JOB END AS JOB,
    COUNT(*) "Total Empl",
    SUM(SAL) "Total Sal"
FROM   EMP, DEPT
WHERE  DEPT.DEPTNO = EMP.DEPTNO
GROUP BY CUBE (DNAME, JOB);
{% endhighlight %}

CUBE는 GROUPING COLUMNS이 가질 수 있는 모든 경우의 수에 대하여 Subtotal을 생성한다.

#### GROUPING SETS 함수
{% highlight sql %}
SELECT 
    CASE GROUPING(DNAME) WHEN 1 THEN 'ALL DEPARTMENTS' ELSE DNAME END AS DNAME,
    CASE GROUPING(JOB) WHEN 1 THEN 'ALL JOBS' ELSE JOB END AS JOB,
    COUNT(*) "Total Empl",
    SUM(SAL) "Total Sal"
FROM   EMP, DEPT
WHERE  DEPT.DEPTNO = EMP.DEPTNO
GROUP BY GROUPING BY (DNAME, JOB);
{% endhighlight %}

GROUPING SETS 함수 사용시 UNION ALL을 사용한 일반 그룹함수를 사용한 SQL과 같은 결과를 얻을 수 있다.
괄호로 묶은 집합별로(괄호 안은 계층 구조가 아닌, 각각의 데이터로 간주) 집계를 구할 수 있다.

---

# 윈도우 함수(WINDOW FUNCTION)

#### WINDOW FUNCTION 개요

행과 행간의 관계를 쉽게 정의하기 위해 만든 함수가 WINDOW FUNCTION이다.
분석 함수나 순위 함수로도 알려져 있는 윈도우 함수는 데이터웨어하우스에서 발전한 기능이다.

* 그룹 내 순위 관련 함수
    * RANK, DENSE_RANK, ROW_NUMBER
* 그룹 내 집계 관련 함수
    * SUM, MAX, MIN, AVG, COUNT
* 그룹 내 행 순서 관련 함수
    * FIRST_VALUE, LAST_VALUE, LAG, LEAD
* 그룹 내 비율 관련 함수
    * CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
* 통계 분석 관련 함수

{% highlight sql %}
-- WINDOW 함수에는 OVER 문구가 키워드로 필수 포함된다.
SELECT WINDOW_FUNCTION (ARGUMENTS) OVER
( [PARTITION BY 칼럼][ORDER BY 절][WINDOWING 절] )
FROM 테이블 명;
{% endhighlight %}

* WINDOW_FUNCTION
    * 기존에 사용하던 함수도 있고, 새롭게 WINDOW 함수용으로 추가된 함수도 있다.
* ARGUMENTS (인수)
    * 함수에 따라 0~N개의 인수가 지정될 수 있다.
* PARTITION BY 절
    * 전체 집합을 기준에 의해 소그룹으로 나눌 수 있다.
* ORDER BY 절
    * 어떤 항목에 대해 순위를 지정할 지 ORDER BY 절을 기술한다.
* WINDOWING 절
    * WINDOWING 절은 함수의 대상이 되는 행 기준의 범위를 강력하게 지정할 수 있다.
    * ROWS는 물리적인 결과 행의 수를, RANGE는 논리적인 값에 의한 범위를 나타낸다.

#### 그룹 내 순위 함수
RANK 함수 
{% highlight sql %}
SELECT JOB, ENAME, SAL,
    RANK() OVER (ORDER BY SAL DESC) ALL_RANK,
    RANK() OVER (PARTITION BY JOB ORDER BY DESC) JOB_RANK
FROM EMP;
{% endhighlight %}
* ORDER BY를 포함한 쿼리문에서 특정 항목(칼럼)에 대한 순위를 구하는 함수이다.
* 이 때 특정 범위(PARTITION) 내에서 순위를 구할 수도 있고, 전체 데이터에 대한 순위를 구할 수도 있다.
* 동일한 값에 대해서는 동일한 순위를 부여하게 된다.

DENSE_RANK 함수
{% highlight sql %}
SELECT JOB, ENAME, SAL,
    RANK() OVER (ORDER BY SAL DESC) ALL_RANK,
    DENSE_RANK() OVER (PARTITION BY JOB ORDER BY DESC) JOB_RANK
FROM EMP;
{% endhighlight %}
* RANK와 흡사하나, 동일한 순위를 하나의 건수로 취급한다.
* RANK의 경우 동일 값이 있으면 같은 순위를 준 후, 동일 값만큼 순위를 건너 뛴다 
    * EX) 1, 2, 2, 4
* DESN_RANK의 경우 동일 값이 있으면 같은 순위를 주고, 다음 순위는 다음 값을 준다.
    * EX) 1, 2, 2, 3

ROW_NUMBER 함수
{% highlight sql %}
SELECT JOB, ENAME, SAL,
    RANK() OVER (ORDER BY SAL DESC) ALL_RANK,
    ROW_NUMBER() OVER (PARTITION BY JOB ORDER BY DESC) JOB_RANK
FROM EMP;
{% endhighlight %}
* 동일 값이라해도 고유한 순위를 부여한다.

#### 일반 집계 함수
SUM 함수
{% highlight sql %}
SELECT MGR, ENAME, SAL, SUM(SAL) OVER (PARTITION BY MGR) MGR_SUM
FROM EMP;
{% endhighlight %}
* 파티션별 윈도우의 합을 구할 수 있다.

MAX 함수
{% highlight sql %}
SELECT MGR, ENAME, SAL, MAX(SAL) OVER (PARTITION BY MGR) MGR_MAX
FROM EMP;
{% endhighlight %}
* 파티션별 윈도우의 최대값을 구할 수 있다.

MIN 함수
{% highlight sql %}
SELECT MGR, ENAME, SAL, MIN(SAL) OVER (PARTITION BY MGR) MGR_MIN
FROM EMP;
{% endhighlight %}
* 파티션별 윈도우의 최소값을 구할 수 있다.

AVG 함수
{% highlight sql %}
SELECT MGR, ENAME, SAL, 
    ROUND (AVG(SAL) OVER (PARTITION BY MGR ORDER BY HIREDATE
         ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)) as MGR_AVG
FROM EMP;
{% endhighlight %}
* AVG 함수와 파티션별 ROWS 윈도우를 이용해 조건에 맞는 데이터에 대한 통계값을 구할 수 있다.

COUNT 함수
{% highlight sql %}
SELECT ENAME, SAL,
    COUNT(*) OVER (ORDER BY SAL
    RANGE BETWEEN 50 PRECEDING AND 150 FOLLOWING) as SIM_CNT
FROM EMP;
{% endhighlight %}
* COUNT 함수와 파티션별 ROWS 윈도우를 이용해 원하는 조건에 맞는 데이터에 대한 통계값을 구할 수 있다.

#### 그룹 내 행 순서 함수
FIRST_VALUE 함수
{% highlight sql %}
SELECT DEPTNO, ENAME, SAL,
    FIRST_VALUE(ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC
    ROWS UNBOUNDED PRECEDING) AS DEPT_RICH
FROM   EMP;

-- RANGE UNBOUNDED PRECEDING
-- 현재 행을 기준으로 파티션 내의 첫 번째 행까지의 범위를 지정한다.
{% endhighlight %}
* 파티션별 윈도우에서 가장 먼저 나온 값을 구한다.

LAST_VALUE 함수
{% highlight sql %}
SELECT DEPTNO, ENAME, SAL
    LAST_VALUE(ENAME) OVER
    (PARTITION BY DEPTNO ORDER BY SAL DESC
    ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS DEPT_POOR
FROM   EMP;

-- ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING;
-- 현재 행을 포함해서 파티션 내의 마지막 행까지의 범위를 지정한다.
{% endhighlight %}
* LAST_VALUE 함수를 이용해 파티션별 윈도우에서 가장 나중에 나온 값을 구한다.

LAG 함수
{% highlight sql %}
SELECT ENAME, HIREDATE, SAL,
    LAG(SAL, 2, O) OVER (ORDER BY HIREDATE) AS PREV_SAL
FROM   EMP
WHERE  JOB = 'SALESMAN';
{% endhighlight %}
* 파티션별 윈도우에서 이전 몇 번째 행의 값을 가져올 수 있다.
 
LEAD 함수
{% highlight sql %}
SELECT ENAME, HIREDATE,
    LEAD(HIREDATE, 1) OVER (ORDER BY HIREDATE) AS "NEXTHIRED"
FROM   EMP;
{% endhighlight %}
* 파티션별 윈도우에서 이후 몇 번째 행의 값을 가져올 수 있다.

#### 그룹 내 비율 함수
RATIO_TO_REPORT 함수
{% highlight sql %}
SELECT ENAME, SAL, ROUND(RATIO_TO_REPORT(SAL) OVER (), 2) AS R_R
FROM   EMP
WHERE  JOB = 'SALESMAN';
{% endhighlight %}
* RATIO_TO_REPORT 함수를 이용해 파티션 내 전체 SUM(칼럼)값에 대한 행별 칼럼 값의 백분율을 소수점으로 구할 수 있다.

PERCENT_RANK 함수
{% highlight sql %}
SELECT DEPTNO, ENAME, SAL
    PERCENT_RANK() OVER (PARTITION BY DEPTNO ORDER BY SAL DESC) as P_R
FROM   EMP;
{% endhighlight %}
* PERCENT_RANK 함수를 이용해 파티션별 윈도우에서 제일 먼저 나오는 것을 0으로, 제일 늦게 나오는 것을 1로 하여, 값이 아닌 행의 순서별 백분율을 구한다.

COUNT_DIST 함수
{% highlight sql %}
SELECT DEPTNO, ENAME, SAL,
    CUME_DIST() OVER (PARTITION BY DEPTNO ORDER BY SAL DESC) as CUME_DIST
FROM   EMP;
{% endhighlight %}
* 파티션별 윈도우의 전체건수에서 현재 행보다 작거나 같은 건수에 대한 누적 백분율을 구한다.

NTILE 함수
{% highlight sql %}
SELECT ENAME, SAL, NTILE(4) OVER (ORDER BY SAL DESC) AS QUAR_TILE
FROM   EMP;
{% endhighlight %}
* NTILE 함수를 이용해 파티션별 전체 건수를 ARGUMENT 값으로 N 등분한 결과를 구할 수 있다.

---

# DCL(DATA CONTROL LANGUAGE)

`DCL`: 유저를 생성하고 권한을 제어할 수 있는 명령어

#### 유저와 권한
{% highlight sql %}
CREATE USER USER1 IDENTIFIED BY PASSWORD1;
GRANT CREATE USER TO USER1;
REVOKE CREATE TABLE FROM USER1;

CREATE ROLE ROLE1;
GRANT CREATE SESSION, CREATE TABLE TO ROLE1;
GRANT ROLE1 TO USER1;
{% endhighlight %}

* CONNECT ROLE
    * ALTER SESSION
    * CREATE CLUSTER
    * CREATE DATABASE LINK
    * CREATE MENU_SEQUENCE
    * CREATE SESSION
    * CREATE SYNONYM
    * CREATE TABLE
    * CREATE VIEW
* RESOURCE ROLE
    * CREATE CLUSTER
    * CREATE INDEXTYPE
    * CREATE OPERATOR
    * CREATE PROCEDURE
    * CREATE MENU_SEQUENCE
    * CREATE TABLE
    * CREATE TRIGGER
    * CREATE

---

# 절차형 SQL

#### PL/SQL

오라클의 PL/SQL은 Block 구조로 되어있고, Block 내에는 DML, Query, IF, LOOP 등을 사용할 수 있으며, 절차적 프로그래밍을 가능하게 하는 트랜잭션 언어이다.
이런 PL/SQL을 이용하여 다양한 저장 모듈(Stored Module)을 개발할 수 있다.
오라클의 저장 모듈에는 Procedure, User Defined Function, Trigger가 있다.

* PL/SQL은 Block 구조로 되어있어 각 기능별로 모듈화가 가능하다.
* 변수, 상수 등을 선언하여 SQL 문장 간 값을 교환한다.
* IF, LOOP등의 절차형 언어를 사용하여 절차적인 프로그램이 가능하도록 한다.
* DBMS 정의 에러나 사용자 정의 에러를 정의하여 사용할 수 있다.
* PS/SQL은 오라클에 내장되어 있으므로, 오라클과 PL/SQL을 지원하는 어떤 서버로도 프로그램을 옮길 수 있다.
* PL/SQL은 응용프로그램의 성능을 향상시킨다.
* PL/SQL은 여런 SQL 문장을 Block으로 묶고 한 번에 Block 전부를 서버로 보내기 때문에 통신량을 줄일 수 있다.

구조는 다음과 같다.

* `DECLEAR`: BEGIN - END 절에서 사용될 변수와 인수에 대한 정의 및 데이터 타입을 선언하는 선언부이다.
* `BEGIN ~ END`: 개발자가 처리하고자 하는 SQL문과 여러가지 비교문, 제어문을 이용하여 필요한 로직을 처리하는 실행부이다.
* `EXCEPTION`: BEGIN - END 절에서 실행되는 SQL 문이 실행될 때 에러가 발생하면 그 에러를 어떻게 처리할 것인지를 정의하는 예외 처리부이다.

#### Procedure
{% highlight sql %}
CREATE [OR REPLACE] Procedure [Procedure_name]
( argument1 [mode] data_type1,
  argument2 [mode] data_type2,
  ... ...)
IS [AS]
  ... ...
BEGIN
  ... ...
EXCEPTION
  ... ...
END;
/
{% endhighlight %}

{% highlight sql %}
DROP Procedure [Prodecure_name];
{% endhighlight %}

{% highlight sql %}
CREATE OR REPLACE Procedure p_DEPT_insert
( v_DEPTNO    in  number,
  v_dname     in  varchar2,
  v_loc       in  varchar2,
  v_result    out varchar2)
IS
cnt number := 0;
BEGIN
  SELECT COUNT(*) INTO CNT
  FROM  DEPT
  WHERE DEPTNO = v_DEPTNO
    AND ROWNUM = 1;
  if cnt > 0 then
    v_result := '이미 등록된 부서번호이다';
  else
    INSERT INTO DEPT (DEPTNO, DNAME, LOC)
    VALUES (v_DEPTNO, v_dname, v_loc);
    COMMIT;
    v_result := '입력 완료';
  end if;
EXCEPTION
  WHEN OTHERS THEN
    ROLLBACK;
   v_result := 'ERROR 발생';
END;
/
{% endhighlight %}

* PL/SQL에서 사용하는 SELECT 문장은 결과값이 반드시 있어야하며, 그 결과 역시 반드시 하나여야 한다.
* 대입 연산자는 `:=`를 사용한다.

{% highlight sql %}
SQL> EXECUTE p_DEPT_insert(10, 'dev', 'seoul', :rslt);
{% endhighlight %}

#### User Defined Function
SUM, SUBSTR, NVL 등의 함수는 벤더에서 미리 만들어둔 내장함수이고, 사용자가 별도의 함수를 만들 수 있다.

{% highlight sql %}
CREATE OR REPLACE Function UTIL_ABS
(v_input in number)
  return NUMBER
IS
  v_return number := 0;
BEGIN
  if v_input < 0 then
    v_return := v_input * -1;
  else
    v_return := v_input;
  end if;
  RETURN v_return;
END;
/
{% endhighlight %}

#### Trigger
트리거란 특정한 테이블에 DML이 수행되었을 때, 데이터베이스에서 자동으로 동작하도록 작성된 프로그램이다.
트리거는 테이블과 뷰, 데티어베이스 작업을 대상으로 정의할 수 있으며, 전체 트랜잭션 작업에 대해 발생되는 트리거와 각 행에 대해서 발생하는 트리거가 있다.
트리거는 데이터베이스에 의해 자동 호출되지만 DML문과 하나의 트랜잭션 안에서 일어나는 일련의 작업들에 포함된다.

{% highlight sql %}
CREATE OR REPLACE Trigger SUMMARY_SALES
  AFTER INSERT       -- 레코드가 입력된 후에
  ON ORDER_LIST      -- ORDER_LIST 테이블에
  FOR EACH NOW       -- 각 ROW마다 트리거 적용
DECLARE
  o_date ORDER_LIST.order_date%TYPE;
  o_prod ORDER_LIST.product%TYPE;
BEGIN
  o_date := :NEW.order_date;
  o_prod := :NEW.product;
  UPDATE SALES_PER_DATE
    SET qty = qty + :NEW.qty,
    amount = amount + :NEW.amount
  WHERE sale_date = o_date
    AND product = o_prod;
  if SQL%NOTFOUNT then
    INSERT INTO SALES_PER_DATE
    VALUES(o_date, o_prod, :NEW.qty, :NEW.amount);
  end if;
END;
/
{% endhighlight %}

* :OLD
    * INSERT: NULL
    * UPDATE: UPDATE되기 전의 레코드 값
    * DELETE: 레코드가 삭제되기 전 값
* :NEW
    * INSERT: 입력된 레코드 값
    * UPDATE: UPDATE된 후의 레코드 값
    * DELETE: NULL



#### 프로시저와 트리거의 차이점

* 프로시저
    * CREATE Procedure 문법 사용
    * EXECUTE 명령어로 실행
    * COMMIT, ROLLBACK 실행 가능
* 트리거 
    * CREATE Trigger 문법 사용
    * 생성 후 자동으로 실행
    * COMMIT, ROLLBACK 실행 안됨
    
---

출처: `"SQL 전문가 가이드, 2013 Edition", 서강수, 한국데이터베이스진흥원, 2013` 

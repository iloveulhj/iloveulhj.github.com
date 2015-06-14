---
layout: post
title:  "[SQL] SQL 활용 #1"
date:   2015-05-31 00:00:00
categories: posts sql
---

이 포스트는 "SQL 전문가 가이드"의 "2-2장. SQL 활용"을 학습한 내용으로 이루어져 있습니다. 

---

# #1 표준 조인

ANSI/ISO SQL에서 표시하는 FROM절의 JOIN 형태는 다음과 같다.

* INNER JOIN
* NATURAL JOIN
* USING 조건절
* ON 조건절
* CROSS JOIN
* OUTER JOIN

---

**INNER JOIN**

INNER JOIN은 JOIN의 DEFAULT 옵션이므로 생략이 가능하다. 
WHERER 절에서 사용하던 JOIN 조건을 FROM 절에서 정의한다. 
USING 조건절이나 ON 조건절을 필수적으로 사용해야 한다.

{% highlight sql %}
-- WHERE 절 JOIN 조건
SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME
FROM   EMP, DEPT
WHERE  EMP.DEPTNO = DEPT.DEPTNO;

-- FROM 절 JOIN  조건
SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME
FROM   EMP INNER JOIN DEPT
ON EMP.DEPTNO = DEPT.DEPTNO;

-- DEFALUT 이므로 옵션 생략 
SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME
FROM   EMP JOIN DEPT
ON     EMP.DEPTNO = DEPT.DEPTNO;
{% endhighlight %}

---

**NATURAL JOIN**

NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다.
NATURAL JOIN이 명시되면 USING 조건절, ON 조건절,  WHERE 절에서 JOIN 조건을 정의할 수 없다.
JOIN에 사용된 칼럼들은 같은 데이터 유형이어야 하고, ALIAS나 테이블명과 같은 접두사는 붙일 수 없다.

{% highlight sql %}
SELECT DEPTNO, EMPNO, ENAME, DNAME
FROM   EMP NATURAL JOIN DEPT;
{% endhighlight %}

와일드카드(*)를 이용하여 SELECT를 하면 NATURAL JOIN은 JOIN의 기준이 되는 칼럼들이 다른 칼럼보다 먼저 출력된다.
그리고 JOIN에 사용된 같은 이름의 칼럼들은 하나로 처리한다.
반면 INNER JOIN의 경우 첫 번째 테이블, 두 번째 테이블의 칼럼 순서대로 데이터가 출력된다. 
JOIN에 사용된 칼럼들도 별개의 칼럼으로 표시한다.

---

**USING 조건절**

NATURAL JOIN에서는 이름이 일치하는 모든 칼럼들에 대해 JOIN이 이루어진다. 
INNJER JOIN의 경우 FROM 절의 USING 조건절을 이용하면 같은 이름을 가진 칼럼들 중에서 원하는 칼럼만 선택적으로 EQUI JOIN을 할 수 있다.
와일드 카드처럼 별도의 칼럼 순서를 지정하지 않으면 USING 조건절의 칼럼을 먼저 출력하고, USING JOIN에 사용된 칼럼을 하나로 처리한다.
USING 조건에 사용된 칼럼에 대해서는 ALIAS나 테이블 이름과 같은 접두사를 붙일 수 없다.

{% highlight sql %}
SELECT * 
FROM   DEPT JOIN DPET_TEMP
USING  (DEPTNO);
{% endhighlight %}

---

**ON 조건절**  

JOIN 서술부(ON 조건절)와 비 JOIN 서술부(WHERE 조건절)을 분리하여 이해가 쉽다.
그리고 칼럼명이 다르더라도 JOIN 조건을 사용할 수 있다. 
USING 절과는 다르게 JOIN에 사용된 칼럼을 SELECT 하는 경우에 ALIAS나 테이블 명과 같은 접두사를 사용하여 SELECT에 사용되는 칼럼을 논리적으로 명확하게 지정해주어야 한다.

ON 조건절에 JOIN 조건 외에도 데이터 검색 조건을 추가할 수 있으나, 검색 목적인 경우는 WHERE 절을 사용할 것을 권고한다.
다만 OUTER JOIN에서 JOIN 대상을 제한하기 위한 목적으로 사용되는 추가 조건의 경우는 ON 절에 표기한다.

{% highlight sql %}
SELECT ST.STADIUM_NAME, SC.STDADIUM_ID, SCHE_DATE, HT.TEAM_NAME,
       AT.TEAM_NAME, HOME_SCORE, AWAY_SCORE
FROM   SCHEDULE SC JOIN STADIUM ST
ON     SC.STADIUM_MD    = ST.STADIUM_ID
       JOIN TEAM HT
ON     SC.HOMETEAM_ID   = HT.TEAM_ID
       JOIN TEAM AT
ON     SC.AWAYTEAM_ID   = AT.TEAM_ID
WHERE  HOME_SCORE       >= AWAY_SCORE +3;      
{% endhighlight %}

---

**CROSS JOIN**

CROSS JOIN은 테이블 간 JOIN 조건이 없는 경우 생길 수 있는 모든 데이터 조합을 말한다.
결과는 양쪽 집합의 M*N 건의 데이터 조합이 발생한다.
정상적인 데이터 모델이라면 CROSS JOIN이 필요한 경우는 많지 않다. 
간혹 튜닝이나 리포트를 작성하기 위해 사용하는 경우가 있을 수 있다. 

{% highlight sql %}
SELECT ENAME, DNAME
FROM   EMP CROSS JOIN DEPT
ORDER  BY ENAME;
{% endhighlight %}

---

**LEFT OUTER JOIN**  
    
조인 수행시 먼저 표기된 좌측 테이블(A)에 해당하는 데이터를 먼저 읽은 후, 나중에 우측 테이블(B)에서 JOIN 대상 데이터를 읽어온다. 
A와 B를 비교해서 B의 JOIN 칼럼에서 같은 값이 있을 때 그 해당 데이터를 가져오고, B의 JOIN 칼럼에서 같은 값이 없는 경우에는 B 테이블에서 가져오는 값들은 NULL 값으로 채운다.
LEFT JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.

{% highlight sql %}
SELECT STADIUM_NAME, STADIDUM.STADIUM_ID, SEAT_COUNT, HOMETEAM_ID, TEAM_NAME
FROM   STADIUM LEFT OUTER JOIN TEAM
ON     STADIUM.HOMETEAM_ID = TEAM.TEAM_ID
ORDER BY HOMETEAM_ID;
{% endhighlight %}

**RIGHT OUTER JOIN**

조인 수행시 LEFT JOIN과 반대로 우측 테이블이 기준이 되어 결과를 생성한다.
A와 B를 비교해서 A의 JOIN 칼럼에서 같은 값이 있을 때 해당 데이터를 가져오고, 
A의 JOIN 칼럼에서 같은 값이 없을 때는  A 테이블에서 가져오는 칼럼들은 NULL 값으로 채운다.
RIGHT JOIN 으로 OUTER 키워드를 생략해서 사용할 수 있다.

{% highlight sql %}
SELECT E.ENAME, D.DEPTNO, D.DNAME, D.LOC
FROM   EMP E RIGHT OUTER JOIN DEPT D
ON     E.DEPTNO = D.DEPTNO;
{% endhighlight %}


**FULL OUTER JOIN**

RIGHT OUTER JOIN과 LEFT OUTER JOIN의 결과를 합집합으로 처리한 결과와 동일하다.
UNION ALL이 아닌 UNION 기능과 같으므로 중복되는 데이터는 삭제된다.
FULL JOIN으로 OUTER 키워들 생랴개헛 사용할 수 있다.

{% highlight sql %}
SELECT *
FROM   DEPT FULL OUTER JOIN DEPT_TEMP;
ON     DEPT.DEPTNO = DEPT_TEMP.DEPTNO;
{% endhighlight %}



---

# #2 집합 연산자

두 개 이상의 테이블에서 조인을 사용하지 않고 연관된 데이터를 조회하는 방법이다.
집합 연산자는 여러 개의 질의의 결과를 연결하여 하나로 결합하는 방식을 사용한다. 
집합 연산자를 사용하기 위해서는 SLELECT 절의 칼럼수가 동일하고, SELECT 절의 동일 위치에 존재하는 칼럼의 데이터 타입이 상호 호환 가능해야 한다.

| 집합 연산자 | 연산자의 의미 |
| :-: | :-- |
| UNION | 여러 개의 SQL문의 결과에 대한 합집합으로 결과에서 모든 중복된 행은 하나의 행으로 만든다. |
| UNION ALL | 여러 개의 SQL문의 결과에 대한 합집합으로 중복된 행도 그대로 결과로 표시된다. 즉, 단순히 결과만 합쳐놓은 것이다. 일반적으로 여러 질의 결과가 상호 베타적일 때 만힝 사용한다. 개별 SQL문의 결과가 서로 중복되지 않는 경우 UNION과 결과가 동일하다. |
| INTERSECT | 여러 개의 SQL문의 결과에 대한 교집합이다. 중복된 행은 하나의 행으로 만든다. |
| EXCEPT | 앞의 SQL문의 결과에서 뒤의 SQL문의 결과에 대한 차집합이다. 중복된 행은 하나의 행으로 만든다. 일부 데이터 베이스는 MINUS를 사용한다. |


{% highlight sql %}
--UNION
SELECT PLAYER_NAME 선수명, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = 'K02'
UNION
SELECT PLAYER_NAME 선수명, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = 'K07'
ORDER BY 1;
{% endhighlight %}

{% highlight sql %}
--UNION ALL
SELECT PLAYER_NAME 선수명, BACK_NO 백넙머
FROM   PLAYER
WHERE  TEAM_ID = 'K02'
UNION ALL
SELECT PLAYER_NAME 선수명, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = 'K07'
ORDER BY 1;
{% endhighlight %}

{% highlight sql %}
--MINUS
SELECT PLAYER_NAME 선수명, BACK_NO 백넙머
FROM   PLAYER
WHERE  TEAM_ID = 'K02'
MINUS
SELECT PLAYER_NAME 선수명, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = 'K07'
{% endhighlight %}

{% highlight sql %}
--INTERECT
SELECT PLAYER_NAME 선수명, BACK_NO 백넙머
FROM   PLAYER
WHERE  TEAM_ID = 'K02'
INTERECT
SELECT PLAYER_NAME 선수명, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = 'K07'
{% endhighlight %}

---

# #3 계층형 질의와 셀프 조인

**계층형 질의** 

테이블에 계층형 데이터가 존재하는 경우 데이터를 조회하기 위해서 계층형 질의를 사용한다.


{% highlight sql %}
-- Oracle 계층형 질의
SELECT ...
FROM 테이블
WHERE condition AND conditions...
START WITH condition
CONNECT BY [NOCYCLE] condition AND condition...
[ORDER SIBLINGS BY column, column, ...]
{% endhighlight %}

- START WITH절은 계층 구조 전개의 시작 위치를 지정하는 구문이다. 즉 루트 데이터를 지정한다.
- CONNECT BY절은 다음에 전개될 자식 데이터를 지정하는 구문이다. 자식 데이터는 CONNECT BY 절에 주어진 조건을 만족해야 한다.(조인)
- PRIOR: CONNECT BY절에 사용되며, 현재 읽은 칼럼을 지정한다.  
  `PRIOR 자식 = 부모 (부모 -> 자식) 순방향 전개`  
  `PRIOR 부모 = 자식 (자식 -> 부모) 역방향 전개`  
- NOCYCLE: 사이클(순환구조 발생)이 발생한 데이터는 런타임 오류가 발생한다. 그렇지만 NOCYCLE를 추가하면 사이클이 발생한 이후의 데이터는 전개되지 않는다.
- ORDER SIBLINGS BY: 형제 노드(동일 LEVEL) 사이에서 정렬을 수행한다.
- WHERE: 모든 전개를 수행한 후에 지정된 조건을 만족하는 데이터만 추출한다.

| 가상 칼럼 | 설명 | 
| :-: | :-: |
| LEVEL | 루트 데이터이면 1, 그 하위데이터이면 2이다. 리프 데이터까지 1씩 증가한다. |
| CONNECT_BY_ISLEAF | 전개 과정에서 해당 데이터가 리프 데이터이면 1, 그렇지 않으면 0이다. |
| CONNECT_BY_ISCYCLE | 전개 과정에서 자식을 갖는데, 해당 데이터가 조상으로서 존재하면 1, 그렇지 않으면 0이다. 여기서 조상이란 자신으로부터 루트까지의 경로에 존재하는 데이터를 말한다. CYCLE 옵션을 사용했을 때만 사용할 수 있다. | 

{% highlight sql %}
SELECT  LEVEL, LPAD(' ', 4 * (LEVEL-1)) || EMPNO 사원,
        MGR 관리자, CONNECT_BY_ISLSEAF ISLEAF
FROM    EMP
START   WITH MGR IS NULL
CONNECT BY PRIOR EMPNO = MGR;
{% endhighlight %}

| 함수 | 설명 |
| :-: | :-: |
| SYS_CONNECT_BY_PATH | 루트 데이터로부터 현재 전개할 데이터까지의 경로를 표시한다. `SYS_CONNECT_BY_PATH(칼럼, 경로분리자)` |
| CONNECT_BY_ROOT | 현재 전개할 데이터의 루트 데이터를 표시한다. `CONNECT_BY_ROOT 칼럼` | 

{% highlight sql %}
SELECT CONNECT_BY_ROOT(EMPNO), SYS_CONNECT_BY_PATH(EMPNO, '/'), EMPNO, MGR
FROM   EMP
START WITH MGR IS NULL
CONNECT BY PRIOR EMPNO = MGR;
{% endhighlight %}

---

**셀프 조인**

셀프 조인이란 동일 테이블 사이의 조인을 말한다. 따라서 FROM 절에 동일 테이블이 두번 이상 나타난다.
동일 테이블 사이의 조인을 수행하면 테이블과 칼럼 이름이 모두 동일하기 때문에 식별을 위해 반드시 테이블 별칭을 사용해야 한다.

{% highlight sql %}
SELECT  E1.EMPNO 사원, E1.MGR 관리자, E2.MGR 차상위_관리자
FROM    EMP E1, EMP E2
WHERE   E1.MGR = E2.EMPNO
ORDER   BY E2.MGR DESC, E1.MGR, E1.EMPNO;
{% endhighlight %}

---

# #4 서브 쿼리

서브쿼리란 하나의 SQL문안에 포함되어 있는 또 다른 SQL문을 말한다. 

- 서브쿼리를 괄호로 감싸서 사용한다.
- 서브쿼리는 단일 행(Single Row) 또는 복수 행(Multiple Row) 비교 연산자와 함께 사용 가능하다.
- 서브쿼리에는 ORDER BY를 사용하지 못한다.

서브쿼리가 SQL문에서 사용이 가능한 곳은 다음과 같다.

- SELECT 절
- FROM 절
- WHERE 절
- HAVING 절
- ORDER BY 절
- INSERT문의 VALUES 절
- UPDATE문의 SET 절

---

**단일 행 서브 쿼리**

서브쿼리가 단일 행 비교 연산자 (=, \<. \<=, >, >=, \<>)와 함께 사용할 때는 서브 쿼리의 결과 건수가 반드시 1건 이하이어야 한다.

{% highlight sql %}
SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버
FROM   PLAYER
WHERE  TEAM_ID = (SELECT TEAM_ID
                  FROM   PLAYER
                  WHERE  PLAYER_NAME = '정남일')
ORDER BY PLAYER_NAME;                  
{% endhighlight %}

---

**다중 행 서브 쿼리**

서브쿼리가 다중 행 비교 연산자 (IN, ALL, ANY, SOME)와 함께 사용해야 한다.

{% highlight sql %}
SELECT REGION_NAME 연고지명, TEAM_NAME 팀명, E_TEAM_NAME 영문팀명
FROM   TEAM
WHERE  TEAM_ID IN (SELECT TEAM_ID
                   FROM PLAYER
                   WHERE PLAYER_NAME = '정현수')
ORDER BY TEAM_NAME;
{% endhighlight %}


---

**다중 칼럼 서브쿼리**

다중 칼럼 서브쿼리의 결과로 여러 개의 칼럼이 반환되어 메인쿼리의 조건과 동시에 비교되는 것을 의미한다.

{% highlight sql %}
SELECT TEAM_ID, PLAYER_NAME, POSITION, BACK_NO, HEIGHT
FROM   PLAYER
WHERE  (TEAM_ID, HEIGHT) IN (SELECT TEAM_ID, MIN(HEIGHT)
                             FROM PLAYER
                             GROUP BY TEAM_ID)
ORDER BY TEAM_ID, PLAYER_NAME;
{% endhighlight %}

---

**연관서브쿼리**

연관 서브쿼리는 서브쿼리 내에 메인칼쿼리 칼럼이 사용된 서브쿼리이다.

{% highlight sql %}
SELECT T.TEAM_NAME, M.PLAYER_NAME, M.POSITION, M.BACK_NO, M.HEIGHT
FROM   PLAYER M, TEAM T
WHERE  M.TEAM_ID = T.TEAM_ID
AND    M.HEIGHT < (SELECT AVG(S.HEIGHT)
                    FROM PLAYER S
                    WHERE S.TEAM_ID = M.TEAM_ID
                    AND S.HEIGHT IS NOT NULL
                    GROUP BY S.TEAM_ID)
ORDER BY M.PLAYER.NAME
{% endhighlight %}

---

* 스칼라 서브쿼리: 한행, 한 칼럼만을 반환하는 서브쿼리를 의미한다.
* 인라인 뷰: FROM 절에서 사용되는 서브쿼리를 인라인 뷰라고 한다.

---

**뷰**

데이터를 가진 테이블과는 달리 뷰는 단지 뷰 정의만을 가지고 있다.
질의에서 뷰가 사용되면 뷰 정의를 참조해서 DBMS 내부적으로 질의를 재작성하여 질의를 수행한다.
 
| 뷰의 장점 | 설명 |
| :-: | :-: |
| 독립성 | 테이블 구조가 변경되어도 뷰를 사용하는 응용 프로그램은 변경하지 않아도 된다. |
| 편리성 | 복잡한 질의를 뷰로 생성함으로써 관련 질의를 단순하게 작성할 수 있다. 또한 해당 형태의 SQL문을 자주 사용할 때 뷰를 이용하면 편리하게 사용할 수 있다. |
| 보안성 | 직원의 급여정보와 같이 숨기고 싶은 정보가 존재한다면, 뷰를 생성할 때 해당 칼럼으 ㄹ빼고 생성함으로써 사용자에게 정보를 감출 수 있다. | 

{% highlight sql %}
-- 뷰 생성
CREATE VIEW V_PLAYER_TEAM AS
SELECT P.PLAYER_NAME, P.POSITION, P.BACK_NO, P.TEAM_ID, T.TEAM_NAME
FROM   PLAYER P, TEAM T
WHERE  P.TEAM_ID = T.TEAM_ID;

-- 뷰 사용
SELECT PLAYER_NAME, POSITION, BACK_NO, TEAM_ID, TEAM_NAME
FROM   V_PLAYER_TEAM
WHERE  PLAYER_NAME LIKE '황%';
{% endhighlight %}

---

출처: `"SQL 전문가 가이드, 2013 Edition", 서강수, 한국데이터베이스진흥원, 2013` 

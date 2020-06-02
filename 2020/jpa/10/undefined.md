# 질문들 정리

### 1. EXISTS \| ALL \| ANY \| SOME \| IN 차이

* 다중 행 연산자\(IN, NOT IN, ANY, ALL, EXISTS\)

![ERD](../../../.gitbook/assets/image%20%2827%29.png)

```sql
--- EMP(사원) DEPT(부서) 테이블 생성

CREATE TABLE DEPT
       (DEPTNO number(10),
        DNAME VARCHAR2(14),
        LOC VARCHAR2(13) );

INSERT INTO DEPT VALUES (10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO DEPT VALUES (20, 'RESEARCH',   'DALLAS');
INSERT INTO DEPT VALUES (30, 'SALES',      'CHICAGO');
INSERT INTO DEPT VALUES (40, 'OPERATIONS', 'BOSTON');

CREATE TABLE EMP (
 EMPNO               NUMBER(4) NOT NULL,
 ENAME               VARCHAR2(10),
 JOB                 VARCHAR2(9),
 MGR                 NUMBER(4) ,
 HIREDATE            DATE,
 SAL                 NUMBER(7,2),
 COMM                NUMBER(7,2),
 DEPTNO              NUMBER(2) );

INSERT INTO EMP VALUES (7839,'KING','PRESIDENT',NULL,'81-11-17',5000,NULL,10);
INSERT INTO EMP VALUES (7698,'BLAKE','MANAGER',7839,'81-05-01',2850,NULL,30);
INSERT INTO EMP VALUES (7782,'CLARK','MANAGER',7839,'81-05-09',2450,NULL,10);
INSERT INTO EMP VALUES (7566,'JONES','MANAGER',7839,'81-04-01',2975,NULL,20);
INSERT INTO EMP VALUES (7654,'MARTIN','SALESMAN',7698,'81-09-10',1250,1400,30);
INSERT INTO EMP VALUES (7499,'ALLEN','SALESMAN',7698,'81-02-11',1600,300,30);
INSERT INTO EMP VALUES (7844,'TURNER','SALESMAN',7698,'81-08-21',1500,0,30);
INSERT INTO EMP VALUES (7900,'JAMES','CLERK',7698,'81-12-11',950,NULL,30);
INSERT INTO EMP VALUES (7521,'WARD','SALESMAN',7698,'81-02-23',1250,500,30);
INSERT INTO EMP VALUES (7902,'FORD','ANALYST',7566,'81-12-11',3000,NULL,20);
INSERT INTO EMP VALUES (7369,'SMITH','CLERK',7902,'80-12-09',800,NULL,20);
INSERT INTO EMP VALUES (7788,'SCOTT','ANALYST',7566,'82-12-22',3000,NULL,20);
INSERT INTO EMP VALUES (7876,'ADAMS','CLERK',7788,'83-01-15',1100,NULL,20);
INSERT INTO EMP VALUES (7934,'MILLER','CLERK',7782,'82-01-11',1300,NULL,10);
commit;
```

![DEPT &#xD14C;&#xC774;&#xBE14;](../../../.gitbook/assets/image%20%2823%29.png)

![EMP &#xD14C;&#xC774;&#xBE14; ](../../../.gitbook/assets/image%20%2828%29.png)

```sql
-- IN 
-- 부서별로 가장 급여를 많이 받는 사원의 정보를 출력
SELECT empno,ename,sal,deptno  
  FROM emp
 WHERE sal IN (SELECT MAX(sal)
                 FROM emp
                GROUP BY deptno);
                
-- '=' 조건을 가지는 경우에 사용
-- IN을 이용해 표현할 수 있는 것은 OR로 표현 가능. OR를 IN으로 표현할때 LIKE는 안됨.                
-- N은 반드시 하나의 컬럼이 비교되어야 한다.
```

![IN &#xACB0;&#xACFC;](../../../.gitbook/assets/image%20%2826%29.png)

```sql
-- ANY 
-- SALESMAN 직업의 급여보다 많이 받는 사원의 사원명과 급여 정보를 출력                                
SELECT ename, sal
  FROM emp
 WHERE deptno != 20
   AND sal > ANY (SELECT sal 
                    FROM emp 
                   WHERE job='SALESMAN');
                   
-- 서브쿼리 결과에서 어느 하나의 값만 만족이 되면 행을 반환           
-- 여기서는 "최솟값"만 비교셈이 된다.
```

![ANY  &#xACB0;&#xACFC;](../../../.gitbook/assets/image%20%2824%29.png)

```sql
-- ALL                               
-- 모든 SALESMAN직업의 급여보다 많이받는 사원의 사원명과 급여정보를 출력
SELECT ename, sal
  FROM emp
 WHERE deptno != 20
   AND sal > ALL (SELECT sal 
                    FROM emp 
                   WHERE job='SALESMAN');               
                   
-- 서브쿼리 결과에서 모든 값 만족이 되면 행을 반환           
-- 여기서는 "최값"만 비교셈이 된다.                         
```

![ALL &#xACB0;&#xACFC; ](../../../.gitbook/assets/image%20%2830%29.png)

```sql
-- EXISTS를 사용안하고 조인으로 사용하게 될 경우... 고비용이 된다. 
SELECT DISTINCT d.deptno, d.dname
  FROM dept d, emp e
 WHERE d.deptno = e.deptno;

-- EXISTS 
-- 사원 최소 한명 이상 있는 부서만을 출력 
SELECT d.deptno, d.dname
  FROM dept d
 WHERE EXISTS 
      (SELECT 1
         FROM emp e
        WHERE e.deptno = d.deptno);
        
-- TRUE,FALSE를 결과로 반환 
-- IN에서는 컬럼을 비교했지만 서브쿼리에서 비교후 boolean을 반환한다.       
-- 첫번째 사원을 만나면, 효율적으 더 이너쿼리를 돌지않고 바로 반환한다. 
```

![EXISTS &#xACB0;&#xACFC;](../../../.gitbook/assets/image%20%2825%29.png)

### 2. n+1 코드 테스트



### 3. EAGER 로딩을 사용중인데도 FETCH JOIN을 사용하는 이유







### 참고자료 

{% embed url="http://www.gurubee.net/lecture/1503" %}








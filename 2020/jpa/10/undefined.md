---
description: 김영한 저 "자바 ORM 표준 JPA 프로그래밍"을 읽고 정리한 내용입니다.
---

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

* 멤버-주문 양방향 연관관계로 설정
* FetchType.EAGER로 설정
* 676 페이지부터 참고하였습니다.

```java
@Entity
public class Member {
@Id @GeneratedValue
   private Long id;

   @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
   private List<Order> orders = new ArrayList<Order>();
}

@Entity
@Table(name = "ORDERS")
public class Order {
   @Id @GeneratedValue
   private Long id;

   @ManyToOne
   private Member member;
}

-- 쿼리가 한번만 나간다
em.find(Member.class, id);

-- 실행된 쿼리 
SELECT M.*, 0.*
 FROM
  MEMBER M
  OUTER JOIN ORDERS 0 ON M.ID=O.MEMBER_ID


-- 문제는 JPQL 쓰면서 발생
-- 즉시 로딩과 지연 로딩에 대해서 신경 쓰지 않고 JPQL만 사용해서 SQL을 생성
List<Member> members =
               em.createQuery("select m from Member m”, Member.class)
               .getResultList();


-- 처음 실행 SQL
SELECT * FROM MEMBER

-- 주문 컬렉션이 즉시로딩인걸 알고 N+1 쿼리 실행
-- 멤버가 3명일 경우 3번 이지만 N명이면 N번 나감
SELECT * FROM ORDERS WHERE MEMBER_ID=1 
SELECT * FROM ORDERS WHERE MEMBER_ID=2 
SELECT * FROM ORDERS WHERE MEMBER_ID=3 

즉, 즉시로딩이 있는 컬렉션에 JPQL을 사용할 때 N+1이 발생한다.
```

* 지연로딩으로 변경할 경우, JPQL 사용을 해도 문제가 없을까? 
* 주문 컬렉션 FetchType.LAZY로 변경

```java
public class Member {
@Id SGeneratedValue
   ...
   @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
   ...order...
}

-- 위와 같은 JPQL을 사용해보면,
-- 이시점의 실행된 SQL은 N+1이 발생하지 않는다.
List<Member> members =
               em.createQuery("select m from Member m", Member.class)
               .getResultList();

-- 실행된 SQL
SELECT * FROM MEMBER

-- 하지만 비즈니스 로직에서 주문컬렉션을 사용해야 될때 N+1이 발생한다.

-- 이경우 까지는 N+1이 발생하지 않는다.
firstMember = members.get(0);
firstMember.getOrders().size(); //지연 로딩 초기화

-- 실행된 SQL
SELECT * FROM ORDERS WHERE MEMBER_ID=?


-- 문제는 다음처럼 모든 "회원에 대해" 연관된 주문 컬렉션을 사용할 때 발생
for (Member member : members) {
   System.out.println( "주문수량=" + member.getOrders().size() );
} 

-- 실행된 SQL
SELECT * FROM ORDERS WHERE MEMBER_ID=1 
SELECT * FROM ORDERS WHERE MEMBER_ID=2 
SELECT * FROM ORDERS WHERE MEMBER_ID=3 

결론, 즉시로딩과 지연로딩 모두 N+1이 발생하였다...
즉시로딩 - 조회시점 발생
지연로딩 - 엔티티 사용시점 발생
```

#### 해결하는 두가지 방법

* 엔티티그래프\(EntityGraph\) 사용하기 
  * 14장에서 자세히 나온다.
  * **FetchType.LAZY** 와 **FetchType.EAGER**로 연관 엔티티를 가져올 것인지를 결정할 수 있다. 하지만 이 구문은 **정적이며 런타임 시 이 설정을 변경하지 못하는 단점**이 있었습니다. **EntityGraph**는 **이러한 점을 보완**하고 연관 엔티티를 어떻게 로딩할 것인지에 대한 정보를 제공함으로서 **엔티티 로딩 속도를 높일 수** 있는 장점이 있습니다. [출처](https://engkimbs.tistory.com/835)
* 페치조인\(fetch join\) 사용하기

### 3. 즉시\(EAGER\) 로딩을 사용중인데도 FETCH JOIN을 사용하는 이유

즉시로딩으로 하든 페치조인을 하든 조회시점에 연관된 데이터를 즉시 가지고 오는것 동일하다. 하지만 즉시로딩을 설정하고 연관된 컬렉션을 가져올 경우 N+1 문제가 발생했다. 페치 조인은 조인쿼리 하나만 나갔다.

조인과 페치조이 차이 먼저  
궁금해서 다시 정리...

```sql
-- 직원만 가져온다.
FROM Employee emp
JOIN emp.department dep

-- 직원과 부서 모두 가져온다. 나중에 부서 테이블을 얻기위해 데이터베이스 접근 필요 X
FROM Employee emp
JOIN FETCH emp.department dep
```



### 참고자료 

{% embed url="http://www.gurubee.net/lecture/1503" %}

{% embed url="https://engkimbs.tistory.com/835" %}

[JPA 설명 유튜브](https://www.youtube.com/watch?v=EwmRIki2HPM&list=PLGTrAf5-F1YLNgq_0TXd9Xu245dJxqJMr&index=65)




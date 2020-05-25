---
description: 김영한 저 "자바 ORM 표준 JPA 프로그래밍"을 읽고 정리한 내용입니다. 최초 작성일  2020.05.18
---

# \#10 객체지향 쿼리 언어 - 발표

## 10.1 객체지향 쿼리

### 미리보기 

```java
------------- 유저이름이 KIM인 멤버만 뽑아내보자. ------------
----------------------------------------------------------
1. JPQL 
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
----------------------------------------------------------
2. Creteria 쿼리 
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq =
	query.select(m).where( cb.equal(m.get("username"), "kim") );
List<Member> resultList = em.createQuery(cq).getResultList();
----------------------------------------------------------
3. QueryDSL 
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;
List<Member> members =
	query.from(member).where(member.username.eq("kim")).list(member);
----------------------------------------------------------
4. Native SQL
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList =
	em.createNativeQuery(sql, Member.class).getResultList();
```

## 10.2 JPQL 

![&#xAE40;&#xC601;&#xD55C;, &#x300C;&#xC790;&#xBC14; ORM &#xD45C;&#xC900; JPA &#xD504;&#xB85C;&#xADF8;&#xB798;&#xBC0D;&#x300D;, &#xC5D0;&#xC774;&#xCF58;, 2015, 354p](../../../.gitbook/assets/image%20%2819%29.png)

### 

#### 시작하기 전에 JPQL의 특징 정리

* 객체지향 쿼리이기 때문에, 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.
* SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.

### 10.2.1 기본 문법과쿼리 API - JPQL

* INSERT문만 은 없다. persist\(\) 사용.
* JPQL에서 UPDATE, DELETE = 벌크 연산, 10.6절에서 설명
* 대소문자 구분, 다만,  SELECT, FROM, AS 같은 JPQL 키워드는 대소문자를 구분 X
* 클래스 명이 아니라 엔티티명
* JPQL은 별칭을 필수\(Alias=Identification variable\)

```sql
SELECT m.username FROM Member m; -- O
SELECT username FROM Member m; -- X
```

#### TypeQuery, Query

* 작성한 JPQL을 실행하려면 쿼리 객체 필요
* 반환할 타입을 명확하게 지정할 수 있으면, TypeQuery 객체
* 지정할 수 없으면, Query 객체를 사용

**TypeQuery 예제**

```java
TypedQuery<Member> query = 
                  em.createQuery ("SELECT m FROM Member m" , Member.class) ;
List<Member> resultList = query.getResultList();
for (Member member : resultList) {
   System.out.println("member = " + member);
}
```

**Query 예제**

* 2개 반환\(username, age\)으로 조회대상 타입이 명확하지 않음.

  ```java
  Query query =
    em.createQuery (" SELECT m.username, m.age from Mennber m") ; 
  List resultList = query.getResultList();
  for (Object o : resultList) {
  Object [] result = (Object []) o; //결과가둘 이상이면 Object [] 반환
  System.out.printin("username = " + result[0]);
  System.out.printin("age = " + result[1]);
  }
  ```

  * 타입을 변환할 필요가 없는 TypeQuery를 사용하는 것이 더 편리
  * **query.getResultList\(\)**: 결과를 예제로 반환. 결과가 없으면 빈 컬렉션을 반환
  * **query.getSingleResult\(\)**: 결과가 정확히 하나일 때 사용

* 결과가 없으면 `NoResultException` 예외가 발생한다.
* 결과과 1 개보다 많으면 `NonUniqueResultException` 예외가 발생한다.

### 10.2.2 파라미터 바인딩 - JPQL

```java
// 이름 기준파라미터
String usernameParam = "User1";
TypedQuery<Member> query =
      em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
query.setParameter("username", usernameParam);
List<Member> resultList = query.getResultList();

// 위치 기준파라미터
List<Member> members =
   em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class)
   .setParameter("username", usernameParam)
   .getResultList();
```

* 이름 기준 파라미터 바인딩 방식을 사용하는 것이 더 명확
* SQL 인젝션 공격 대응책
  * 위험코드 "select m from Member m where m.username = '" + usernameParam + "'"

### 10.2.3  프로젝션 - JPQL

* 조회할 대상을 지정하는 것
* 프로젝션 대상인 3가지 엔티티，엠비디드 타입，스칼라 타입 그리고 3가지의 복합과 NEW 명령어에 대해 알아보자.

#### **엔티티**

* 엔티티를 프로젝션 대상으로 사용
* 조회한 엔티티는 영속성 컨텍스트에서 관리

  ```sql
  SELECT m FROM Member m //회원
  SELECT m.team FROM Member m //팀
  ```

#### **엠비디드**

* 임베디드 타입은 엔티티 타입이 아닌 값 타입
* 영속성 컨텍스트에서 관리 X
* 조회의 시작점이 될 수 없다는 제약

```java
String query = "SELECT a FROM Address a";
String query = "SELECT o.address FROM Order o"; 

List addresses = em.createQuery(query, Address.class).getResultList();
```

#### **스칼라** 

* 숫자，문자，날짜와 같은 기본 데이터 타입

```java
// 1
List<String> usernames = 
                  em.createQuery("SELECT username FROM Member m", String.class)
                  .getResultList();

// 2 통계쿼리
Double orderAmountAvg = 
            em.createQuery("SELECT AVG(o.orderAmount) FROM Order o", Double.class)
            .getSingleResult();
```

#### **여러값 조회**

```java
// 1 
List<Object[]> resultList = 
                        em.createQuery("SELECT m.username, m.age FROM Member m")
                        .getResultList();
for (Object[] row : resultList) {
   String username = (String)row[0];
   Integer age = (Integer)row[1];
}

// 2 
List<Object[]> resultList =
         em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o")
         .getResultList ();
for (Object[] row : resultList) {
   Member member = (Member)row[0]; //엔티티
   Product product = (Product)row[1]; //엔티티
   int orderAmount = (Integer)row[2]; //스칼라
}
```

* 이때도 조회한 엔티티는 영속성 컨텍스트에서 관리

**NEW 명령어**

* 실제 개발시 Object\[\] 직접 사용 X, UserDTO 처럼 의미 있는 객체로 변환해서 사용

```java
//사용전 
List resultList = em.createQuery("SELECT m.username, m.age FROM Member m")                  
                 .getResultList();

//객체 변환 작업
List userDTOs = new ArrayList();
for (Object[] row : resultList) {
   String username = (String) row[0];
   Integer age = (Integer) row[1];
}
return userDTOs;

// 사용후 
TypedQuery query = 
            em.createQuery( 
             "SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m"
            , UserDTO.class); 
List resultList = query.getResultList();
```

* TypeQuery 사용할 수 있어서 지루한 객체 변환 작업을 줄일 수 있다.

#### 주의사항

1. 패키지 명을 포함한 전체 클래스 명을 입력해야 한다.      \(new jpabook.jpql.UserDTO\(m.username, m.age\)\)
2. 순서와 타입이 일치하는 생성자가 필요하다.

```java
public class UserDTO {

   private String username;
   private int age;

   public UserDTO(String username, int age) {
      this.username = username;
      this.age = age;
   }
   //...
}
```

### 10.2.4 페이징 API - JPQL

* 데이터베이스마다 페이징을 처리하는 SQL 문법이 다르다. \(오라클, SQLServer 너무 어렵다. 365페이지 참조\)

```sql
//MySQL
SELECT
   M.ID AS ID,
   M.AGE AS AGE,
   M.TEAM_ID AS TEAM_ID,
   M.NAME AS NAME
FROM
   MEMBER M
ORDER BY
   M.NAME DESC LIMIT ?, ?
 
 //오라클
SELECT *
FROM
( SELECT ROW_.*, ROWNUM ROWNUM_
   FROM
   ( SELECT 
         M.ID AS ID,
         M.AGE AS AGE,
         M.TEAM_ID AS TEAM_ID,
         M.NAME AS NAME
     FROM MEMBER M
     ORDER BY M.NAME
   ) ROW_
   WHERE ROWNUM <= ?
)
WHERE ROWNUM_ > ?
```

* ?에 바인딩하는 값도 데이터베이스마다 다르다.

#### 페이징 API 코드 \(멤버 조회해서 11~30번째 가져오기\)

```java
TypedQuery<Member> query =
   em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC", Member.class);
query.setFirstResult(10); //시작위치
query.setMaxResults(20); // 데이터수
query.getResultList();
```

* 페이징 SQL 더 최적화하려면 Native SQL 사용

### 10.2.5 집합과 정렬 - JPQL

**집합함수 사용**

```sql
select
   COUNT (m) , //회원수
   SUM(m.age), //나이합
   AVG(m.age), //의롸01
   MAX (m.age), //최대 나이
   MIN (m. age) //최소 나이
from Member m
```

| 함수 | 설명 |
| :--- | :--- |
| COUNT | 결과 수를 구한다. 반환 타입: Long |
| MAX, MIN | 최대, 최소 값을 구한다. 문자, 숫자, 날짜 등에 사용한다. |
| AVG | 평균값을 구한다. 숫자타입만 사용할 수 있다. 반환 타입: Double |
| SUM | 합을 구한다. 숫자타입만 사용할 수 있다. 반환 타입: 정수합 Long, 소수합: Double, Biginteger합: Biginteger, BigDecimal합: BigDecimal |

**참고사항**

* NULL 값은 무시하므로 통계에 잡히지 않는다\(DISTINCT가 정의되어 있어도 무시된다\).
* 만약 값이 없는데 SUM, AVG, MAX, MIN 함수를 사용하면 NULL 값이 된다. 단, COUNT는 0이 된다.
* DISTINCT를 COUNT에서 사용할 때 임베디드 타입은 지원하지 않는다.

**GROUP BY, HAVING**

* 팀 이름을 기준으로 그룹별로 묶어서 통계

  ```sql
  SELECT t.name, COUNT(m.age), SUM(m.age), AVG(m.age), MAX(m.age), MIN(m.age)
  FROM Member m LEFT JOIN m.team t
  GROUP BY t.name
  ```

* 그룹별 통계 데이터 중에서 평균나이가 10살 이상인 그룹을 조회

  ```sql
  SELECT t.name, COUNT( m.age ), SUM( m.age ), AVG( m.age ), MAX( m.age ), MIN(m.age)
  FROM 
   Member m LEFT JOIN m.team t
  GROUP BY t.name
  HAVING AVG(m.age) >= 10
  ```

* 보통 전체 데이터를 기준으로 처리하므로 실시간으로 사용하기엔 부담
* 사용자가 적은 새벽에 통계 쿼리를 실행

**ORDER BY**

* 결과를 정렬

  ```sql
  SELECT t.name, COUNT(m.age) as ent
  FROM Member m LEFT JOIN m.team t
  GROUP BY t.name
  ORDER BY ent
  ```

### 10.2.6 JPQL 조인 - JPQL

```sql
1. 내부조인 
SELECT m FRCM Member m INNER JOIN m. team t WHERE t.name = :teamName
2. 외부조인
SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
3. 컬렉션조인 - 일대다 관계, 다대다 관계 조인
SELECT t, m FROM Team t LEFT JOIN t.members m --일대다
4. 세타조인  
SELECT count(m) FROM Member m, Team t WHERE m.username = t.name
```

```java
String teamName = "팀A";
String query = "여기 원하는 조인 쿼리를 넣는다." 
                     + "WHERE t.name = :teamName";
List<Member> members = em.createQuery(query, Member.class)
                        .setParameter("teamName", teamName)
                        .getResultList();
```

* JPA 2.1 부터 조인시 ON 절 지원.

```sql
SELECT m,t FROM Member m LEFT JOIN m.team t ON t.name = 'A'
```

### 10.2.7 페치 조인 - JPQL

* SQL 조인의 종류는 아니고 JPQL에서 성능 최적화를 위해 제공하는 기능
* 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능

**엔티티 페치 조인**

```sql
//회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회
SELECT m FROM Member m JOIN FETCH m.team
```

아래 설명할 정도로 책이 깔끔하지안은거 가타

**컬렉션페치조인**

**페치 조인과 DISTINCT**

Q. distinct는 컬렉션 패치조인하고 꼭 가이 써야 하는가?

**페치 조인의 특징과 한계**

* 특징
  * 페치 조인은 글로벌 로딩 전략보다 우선한다\(`@0neToMany(fetch = FetchType.LAZY)` //글로벌 로딩 전략\)
  * 글로벌 로딩 전략을 지연 로딩으로 설정해도 JPQL에서 페치 조인을 사용하면 페치 조인을 적용해서 함께 조회
  * 따라서 지연 로딩을 사용하고 최적화가 필요하면 페치 조인을 적용하는 것이 효과적
* 한계
  * 페치 조인 대상에는 별칭을 줄 수 없다. 따라서 SELECT, WHERE 절，서브 쿼리에 페치 조인 대상을 사용할 수 없다. \(하이버네이트는 지원\)
  * 둘 이상의 컬렉션을 페치할 수 없다. 카테시안 곱이 만들어지므로 주의. 
  * 컬렉션을 페치 조인하면 페이징 API（setFirstResult, setMaxResults）를 사용할 수 없다. 컬렉션\(일대다\)이 아닌 단일 값 연관 필드（일대일，다대일）들은 페치 조인을 사용해도 페이징 API를 사용할 수 있다.

**페치조인 정리**

* 페치 조인은 실무에서 자주 사용 
* 객체 그래프를 유지할 때 사용하면 효과적?
* 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하다면 억지로 페치 조인을 사용하기보다는 여러 테이블에서 필요한 필드들만 조회해서 DTO로 반환하는 것이 더 효과적일 수 있다.

### 10.2.8 경로표현식 - JPQL

* JPQL에서 사용하는 경로 표현식을 알아보고 경로 표현식을 통한 묵시적 조인도 알아보자.
* 경로 표현식이라는 것은 .\(점\)을 찍어 객체 그래프를 탐색하는 것
* m.username, m.team, m.orders, t.name이 경로 표현식

  ```sql
  SELECT m.username
  FROM Member m
      JOIN m.team t
      JOIN m.orders o
  WHERE t.name = '팀A'
  ```

  경로 표현식을 이해하려면 우선 다음 용어들을 알아야 한다. 

* 상태필드: 단순히 값을 저장하는 필드
* 연관필드: 객체 사이의 연관관계를 맺기 위해 사용하는 필드, 임베디드 타입 포함
  * 단일 값 연관 필드: @ManyToOne, @OneToOne, 대상이 엔티티
  * 컬렉션 값 연관 필드: @OneToMany, @ManyToMany, 대상이 컬렉션

```java
@Entity
public class Member {

   @Id @GeneratedValue
   private Long id; 

   @Colunm (name = "name")
   private String username; //상태 필드
   private Integer age; //상태 필드

   @ManyToOne(..)
   private Team team; //연관 필드 (단일 값 연관 필드)
   @OneToMany(..)
   private List<Order> orders; //연관 필드 (컬렉션 값 연관 필드)
   //...
}
```

#### 경로표현식 특징

* 상태 필드 경로: 경로 탐색의 끝이다. 더는 탐색할 수 없다.
* 단일 값 연관 경로: 묵시적으로 내부 조인이 일어난다. 단일 값 연관 경로는 계속 탐색할 수 있다.
* 컬렉션 값 연관 경로: 묵시적으로 내부 조인이 일어난다. 더는 탐색할 수 없다. 단, FROM 절에서 조인을 통해 별칭을 얻으면 별칭으로 탐색할 수 있다.

```sql
// 1. 명시적 조인
SELECT m FROM Member m JOIN m.team t
// 2. 묵시적 조인 - 경로 표현식에 의해 묵시적으로 조인이 일어나는 것，내부 조인(INNER JOIN)만 할 수 있다.
SELECT m.team FROM Member m
```

#### 상태 필드 경로탐색

```sql
// JPQL - 상태 필드 경로
SELECT m.username, m.age FROM Member m
// SQL - 상태 필드 경로
SELECT m.name, m.age FROM Member m
```

**단일 값 연관 경로**

```sql
JPQL - 단일 값 연관 경로
SELECT o.member FROM Order o -- order:member N:1
SQL - 단일 값 연관 경로
SELECT m.*
FROM Orders o INNER JOIN Member m on o.member_id=m.id
```

단일 값 연관 경로 실습 - 354p UML 참고

* **주문** 중에서 **상품명**이 'productA'고 **배송지**가 'JINJU'인 회원이 소속된 팀을 조회해보자.

```sql
SELECT o.member.team
FROM Order o
WHERE o.product.name = 'productA' and o.address.city = 'JINJU'
```

* 문제 - 위에는 명시적일까? 묵시적걸까?
  * o.member.tea와 같이 경로탐색을 사용했기에 묵시적이다.
* 문제 - 위 쿼리는 몇번의 조인이 일어날까?
  * 3번 조인 - 아래 참고 

```sql
// 실행된 SQL
SELECT t.*
FROM Orders o
INNER JOIN Member m ON o.member_id=m.id
INNER JOIN Team t ON m.team_id=t.id
INNER JOIN Product p ON o.product_id=p.id
WHERE p.name='productA' AND o.city='JINJU'
```

#### 컬렉션 값 연관 경로 탐색

* 컬렉션 값에서 경로 탐색을 시도하는 실수를 많이 한다.

```sql
SELECT t.members FROM Team t //성공
SELECT t.members.username FROM Team t //실패
```

* t.members 처럼 컬렉션까지는 경로 탐색이 가능하나 username처럼 컬렉션에서 경로 탐색을 시작하는 것은 허락하지 않는다. username을 얻기 위해선 아래 같이 각팀 멤버들을 별칭으로 받는다.

```sql
//별칭을 얻은 뒤 부터 탐색 다시 가능 
SELECT  m.username FROM Team t JOIN t.members m
```

size라는 특별한 기능을 사용 가능 

```sql
// 이런 기능이 있다
SELECT t.members.size FROM Team t
```

#### **묵시적 조인 시 주의사항**

경로 탐색을 사용하면 묵시적 조인이 발생해서 SQL에서 내부 조인이 일어난다.

* 항상 내부 조인이다.
* 컬렉션은 경로 탐색의 끝이다. 컬렉션에서 경로 탐색을 하려면 명시적으로 조인해서 별칭을 얻어야 한다.
* 경로 탐색은 주로 SELECT, WHERE 절 \(다른 곳에서도 사용됨\)에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM 절에 영향을 준다.

결론, **단순하고 성능에 이슈가 없으면** 크게 문제가 안 되지만 성능이 중요하면 분석하기 쉽도록 묵시적 조인보다는 **명시적 조인을 사용하자.**

### 10.2.9 서브 쿼리 - JPQL

* JPQL도 SQL처럼 서브 쿼리를 지원
* WHERE, HAVING 절에서만 사용 O, 
* SELECT, FROM 절에서는 사용 X

```sql
나이가 평균보다 많은 회원 찾는 쿼리 
SELECT m FROM Member m
WHERE m.age > (SELECT avg(m2.age) FROM Member m2)
```

한 건이라도 주문한 고객을 찾는 쿼리 

```sql
SELECT m FROM Member m
WHERE (SELECT FROM (o) from Order o WHERE m = o.member) > 0

-- size 기능 활용하면 위와 동일한 결과 
SELECT m FROM Member m WHERE m.orders.size > 0
```

####  서브 쿼리 함수

* \[NOT\] EXISTS \(subquery\)
* {ALL \| ANY \| SOME} \(subquery\)
* \[NOT\] IN \(subquery\)

서브쿼리들을 실습해보자. 

```sql
문제 - 팀A 소속인 회원 
요구 - 서브쿼리 사용하고 EXISTS 사용

SELECT m FROM Member m
WHERE EXISTS (SELECT t FROM  m.team t WHERE t.name = '팀A*)
```

```sql
문제 - 전체 상품 각각의 재고보다 주문량이 많은 주문들 
요구 - 서브쿼리 사용하고 ALL, ANY, SOME 중 하나 사용

SELECT o FROM Order o
WHERE o.orderAmount > ALL (SELECT p.stockAmount FROM Product p)
```

```sql
문제 - 어떤 팀이든 팀에 소속된 회원 
요구 - 서브쿼리 사용하고 ALL, ANY, SOME 중 하나 사용

SELECT m FROM Member m
WHERE m.team = ANY (SELECT t FROM Team t)
```

```sql
문제 - 20세 이상을 보유한 팀 
요구 - 서브쿼리 사용하고 IN 사용

SELECT t FROM Team t
WHERE t IN (SELECT t2 From Team t2 JOIN t2.members m2 WHERE m2.age >= 20)
```

### 10.2.10 조건식 - JPQL

```sql
문제 - WHERE절의 like 조건에서 '회원%'는 회원1, 회원ABC같은 것을 찾아낸다. 
       그럼 '회원%'와 같이 진짜 특수문자는 어떻게 찾을까?

WHERE m.username LIKE '회원\%' ESCAPE '\'
```

#### 컬렉션식

컬렉션 식은 컬렉션에만 사용하는 특별한 기능이다. 문제 - 주문이 하나라도 있는 회원 조회

```sql
// JPQL 
// empty는 빈컬렉션 비교식 
SELECT m FROM Member m
WHERE m.orders IS NOT EMPTY

// 실행된 SQL 
SELECT m.* FROM Member m
WHERE 
   exists (
      SELECT o.id
      FROM Orders o
      WHERE m.id=o.member_id
   )
```

아래는 가능할까?

```sql
SELECT m FROM Member m
WHERE m.orders IS NULL
```

* IS NULL은 컬렉션 식이 아니다.

**컬렉션 멤버식**

매개변수로 받은 멤버가 속한 팀을 컬렉션의 멤버식을 이용해 찾아보자.

```sql
SELECT t FROM Team t
WHERE :memberParam member of t.members
```

**스칼라식**

* 문자함수: CONCAT, LOWER, UPPER, LENGTH, SUBSTRING, TRIM, LOCATE   
  아래의 값은 어떻게 나올까?

  ```sql
  SUBSTRING('ABCDEF', 2,3) = 답은?
  TRIM(' ABC ')  = 답은?
  LOCATE('DE', 'ABCDEFG') = 답은?
  ```

* 수학함수: ABC, SQRT, MOD, SIZE, INDEX
* 날짜함수: CURRENT\_DATE, CURRENT\_TIME, CURRENT\_TIMESTAMP

하이버네이트에서는 아래 기능 지원

```sql
SELECT year(CURRENT_TIMESTAMP), month(CURRENT_TIMESTAMP), day(CURRENT_TIMESTAMP)
FROM Member
```

**CASE 식**

```sql
멤버의 나이가 10세 이하면 '학생요금' 60세 이상이면 '경로요금'
1. 기본 CASE
SELECT 
   CASE WHEN m.age <= 10 THEN '학생요금'
        WHEN m.age >= 60 THEN '경로요금'
        ELSE '일반요금'
   END
FROM Member m

팀A일 경우 인센티브 110, 팀B일 경우 120, 이외에는 105
2. 심플 CASE
SELECT 
   CASE t.name
      WHEN '팀A' THEN '인센티브110%'
      WHEN  '팀B' THEN '인센티브 120%'
      ELSE '인센티브 105%'
   END
FROM Team t

멤버의 이름값이 NULL이면 '이름 없는 회원'을 반환 
3. COALEASCE
SELECT COALESCE(m.username, '이름 없는 회원') FROM Member m

관리자면 NULL을 반환하고 나머지는 본인의 이름 반환 
4. NULLIF
SELECT NULLIF(m.username, '관리자') FROM Member m
```

### 10.2.11 다형성쿼리 - JPQL

JPQL로 부모 엔티티를 조회하면 그 자식 엔티티도 함께 조회한다.

```java
@Entity
@InherItance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {...}

@Entity
@DiscriminatorValue("B")
public class Book extends Item {
   //...
   private String author;
}
//Album, Movie 생략
```

```java
List resultList = em.createQuery("select i from Item i").getResultList();
```

위 실행시 두가지 전략\(단일테이블, 조인\)을 사용할때 SQL이 다음과 같이 나온다.

```sql
// InheritanceType.SINGLE_TABLE 때 SQL
SELECT * FROM ITEM

// InheritanceType.JOINED 때 SQL
SELECT
   i.ITEM_ID, i.DTYPE, i.name, i.price, i.stockQuantity,
   b.author, b.isbn,
   a.artist, a.etc,
   m.actor, m.director
FROM 
   Item i
LEFT OUTER JOIN 
   Book b ON i.ITEM_ID=b.ITEM_ID
LEFT OUTER JOIN 
   Album a ON i.ITEM_ID=a.ITEM_ID
LEFT OUTER JOIN 
   Movie m ON i.ITEM_ID=m.ITEM_ID
```

`TYPE` - 조회 대상을 특정 자식 타입으로 한정할 때 사용 예, Item 중에 Book, Movie를 조회하라.

```sql
// JPQL
SELECT i from Item i
where type(i) IN (Book, Movie)

// SQL
SELECT i FROM Item i
WHERE i.DTYPE in ('B', 'M')
```

TREAT - 자바의 타입 캐스팅과 비슷하며 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다. 부모 타입인 Item을 자식 타입인 Book으로 다룬다. 따라서 author 필드에 접근할 수 있다.

```sql
//JPQL
SELECT i FROM Item i WHERE TREAT(i as Book).author = 'kim'

//SQL
SELECT i.* FROM Item i
where i.DTYPE='B' AND i.author='kim'
```

### 10.2.12 사용자 정의 함수 호출\(JPA 2.1\) - JPQL

* 문법: function\_invocation::= FUNCTION\(function\_name {, function\_arg}\*\)
* 예: SELECT function\('group\_concat', i.name\) FROM Item i // 인터넷 더 찾아보자

**하이버네이트의 경우** - 아래같이 방언 클래스를 상속해서 구현하고 사용할 데이터베이스 함수를 미리 등록해야 한다.

```java
// 방언 클래스를 상속
public class MyH2Dialect extends H2Dialect {
   public MyH2Dialect() {
      registerFunction( "group_concat", new StandardSQLFunction
         ("group_concat", StandardBasicTypes. STRING));
   }
}

//상속한 방언 클래스 등록(persistence.xml)
<property name="hibernate.dialect" value=”hello.MyH2Dialect" />

// 이렇게 사용 
SELECT group_concat(i.name) FROM Item i
```

### 10.2.13 기타정리 - JPQL

* enum은 = 비교 연산만 지원한다.
* 임베디드 타입은 비교를 지원하지 않는다
* JPA 표준은 ''을 길이 0인 EmptyString으로 정했지만 데이터베이스에 따라 ''를 NULL로 사용하는 데이터베이스도 있으므로 확인하고 사용하자.
* NULL 정의
  * NULL과의 모든 수학적 계산 결과는 NULL
  * Null == Null은 알 수 없는 값
  * Null is Null은 참

JPA 표준 명세는 Null\(U\) 값과 TRUE\(T\), FALSE\(F\)의 논리 계산을 다음과 같이 정의하였다.  
나중에 자세히 알아보기

**AND**

| AND | T | F | U |
| :--- | :--- | :--- | :--- |
| T | T | F | U |
| F | F | F | F |
| U |  |  | UFU |

**OR**

| OR | T | F | U |
| :--- | :--- | :--- | :--- |
| T | T | T | T |
| F | T | F | U |
| U | T | U | U |

**NOT**

| NOT |  |
| :--- | :--- |
| T | F |
| F | T |
| U | U |

### 10.2.14 엔티티 직접 사용 - JPQL

#### 기본키값

JPQL에서 엔티티 객체를 직접 사용하면 SQL에서는 해당 엔티티의 기본 키값을 사용한다

```sql
SELECT count (m. id) FROM Member m //엔티티의 아이디를 사용
SELECT count (m) FROM  Member m //엔티티를 직접 사용

// 위 둘다 같은 쿼리 
SELECT count(m.id) as ent FROM Member m
```

엔티티를 파라미터로 직접 받는 코드\(엔티티로 파라미터 줘도 ID로 조회\)

```java
String qlString = "SELECT m FROM Member m WHERE m = :member";
List resultList = em.createQuery(qlString)
                  .setParameter("member", member)
                  .getResultList();

// 실행된 SQL 
SELECT m.* FROM Member m WHERE m.id=?

// qlString 쿼리를 ID로 조회해도 같다.
"SELECT m FROM Member m WHERE m = :member";
```

#### 외래키값

```java
// 외래 키 대신에 엔티티를 직접 사용
Team team = em.find(Team.class, 1L); // 기본키 값이 1L인 팀 엔티티를 파라미터로 사용
String qlString = "SELECT m FROM Member m WHERE m.team = :team";
List resultList = em.createQuery(qlString)
                  .setParameter("team", team)
                  .getResultList();

// 실행된 SQL
SELECT m.* from Member m WHERE m. team_id=?

// qlString 쿼리를 ID로 조회해도 같다.
"SELECT m FROM Member m WHERE m.team.id = :teamld"
```

### 10.2.15 Named Query: 정적쿼리 - JPQL

JPQL 쿼리는 크게 동적 쿼리와 정적 쿼리로 나뉜다.

* **동적쿼리**
  * em.createQuery\("select.."\) 처럼 JPQL을 문자로 완성해서 직접 넘기는 것을 동적 쿼리라 한다. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성할 수 있다.
* **정적쿼리** = Named 쿼리
  * 미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용한다. 한 번 정의하면 변경할 수 없는 정적인 쿼리다.
  * **애플리케이션 로딩시점에 JPQL 문법을 체크**하고 미리 파싱해 둔다. 따라서 **오류를 빨리 확인**할 수 있고，사용하는 시점에는 **파싱된 결과를 재사용**하므로 성능상 이점도 있다. 그리고 Named 쿼리는 변하지 않는 정적 SQL이 생성되므로 **데이터베이스의 조회 성능 최적화에도 도움**이 된다.

**`@NamedQuery` `@NamedQueries`**

```java
// 1. NamedQuery 설정  
@Entity
@NamedQuery(
   name = "Member.findByUsername",
   query= "SELECT m FROM Member m WHERE m.username = :username" )
public class Member {
   // ...
}

// 2. NamedQueries 복수개 설정
@Entity
@NamedQueries({
   @NamedQuery(
      name = "Member.findByUsername",
      query="SELECT m FROM  Member m WHERE m.username = :username"),
   @NamedQuery(
      name = "Member.count",
      query= "SELECT count(m) FROM Member m")
   })
public class Member {
   // ...
}

// 3. 사용시 
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                        .setParameter("username", ”회원1")
                        .getResultList();
```

**Member.findByUsername**처럼 앞에 엔티티 이름을 주었는데, Named 쿼리는 **영속성 유닛 단위로 관리**되므로 **충돌을 방지**하기 위해 엔티티 이름을 앞에 주었다. 즉, 엔티티 이름이 앞에 있으면 관리하기가 쉽다.

```java
@Target({TYPE})
public @interface NamedQuery {
   String name (); //Named 쿼리 이름 (필수)
   String query () ; //JPQL 정의 (필수)
   LockModeType lockMode () default NONE; //쿼리 실행 시 락모드를
   //설정할 수 있다.
   QueryHint [ ] hints () default {}; //JPA 구현체에 쿼리 힌트를 줄 수 있다.
}
```

* **`lockMode`**: 쿼리 실행 시 락을 건다. 16.1절에서 자세히.
* **`hints`**: SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트. 2차 캐시를 다룰 때 사용한다.

#### Named 쿼리를 XML에 정의

* JPA에서 어노테이션으로 작성할 수 있는 것은 XML로도 작성
* Named 쿼리를 작성할 때, XML을 사용하는 것이 더 편리. 멀티라인을 자바에서 다루면은 아래처럼 상당히 귀찮기 때문이다.

```java
// 자바 
"SELECT n +
"CASE t.name WHEN '팀A' THEN '인센티브 110%' " +
"WHEN '팀B' THEN，'인센티브120%' " +
"ELSE '인센티브 105%' END " +
"from Team t";

// 구루비 (비교대상)
'''
   SELECT 
   CASE t.name WHEN '팀A' THEN '인센티브110%'
   WHEN '팀B' THEN '인센티브 120%'
   ELSE '인센티브105%，END
   FROM Team t
'''
```

```markup
<!-- META-INF/ormMember.xml 생성 -->
<?xml version:"1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">

   <named-query name= "Member.findByUsername">
      <query><CDATA[
         SELECT m
         FROM Member m
         WHERE  m.username = :username
      ]></query>
   </named-query>

   <named-query name="Member.count">
      <query>SELECT count(m) FROM Member m</query>
   </named-query>

</entity-mappings>
```

* 참고
  * XML에서 &, &lt;,&gt;는 XML 예약문자라 &, &lt;, &gt;를 사용해야 하지만 &lt;!\[CDATA\[ \]\]&gt;안에서는 예약문자 자유롭게 사용.
  * XML과 어노테이션에 같은 설정 이 있으면 XML이 우선권

## 10.3 Criteria 쿼리

### 10.3.1 기초 - Criteria

### 10.3.2 쿼리 생성 - Criteria

### 10.3.3 조회 - Criteria

### 10.3.4 집합 - Criteria

### 10.3.5 정렬 - Criteria

### 10.3.6 조인 - Criteria

### 10.3.7 서브쿼리 - Criteria

### 10.3.8 IN 식 - Criteria

### 10.3.9 CASE 식 - Criteria

### 10.3.10 파라미터 정의 - Criteria

### 10.3.11 네이티브 함수 호출 - Criteria

### 10.3.12 동적 쿼리 - Criteria

### 10.3.13 함수정리 - Criteria

### 10.3.14 메타모델 API - Criteria

## 10.4 QueryDSL

### 10.4.1 설정 - QueryDSL

### 10.4.2 시작 - QueryDSL

### 10.4.3 검색조건 쿼리 - QueryDSL

### 10.4.4 결과 조회 - QueryDSL

### 10.4.5 페이징과 정렬 - QueryDSL

### 10.4.6 그룹 - QueryDSL

### 10.4.7 조인 - QueryDSL

### 10.4.8 서브 쿼리 - QueryDSL

### 10.4.9 프로젝션과 결과 반환  - QueryDSL

### 10.4.10 수정, 삭제 배치 쿼리  - QueryDSL

### 10.4.11 동적 쿼리 - QueryDSL

### 10.4.12 메소드 위임 - QueryDSL

### 10.4.13 정리 - QueryDSL

## 10.5 Native SQL

### 10.5.1 사용 - Native SQL

### 10.5.2 Named Native SQL

### 10.5.3 XML에 정의

### 10.5.4 정리

### 10.5.5 Stored Procedure\(JPA 2.1\)

## 10.6 객체지향 쿼리 심화

### 10.6.1 벌크연산

### 10.6.2 영속성컨택스트와 JPQL

### 10.6.3 find\(\) vs JPQL

### 10.6.4 JPQL과 플러시 모드

## 10.7 정리

### :star: 정리

* JPQL은 SQL을 추상화해서 특정 DB에 종속되지 않는다.
* Creteria나 QueryDQL은 JPQL을 만들어주는 비럳 역할을 할 뿐이므로 JPQL을 잘 알아야 한다.
* Creteria나 QueryDQL 를 사용하면 동적으로 변하는 쿼리 작성 가능
* Creteria  -&gt; JPA지원 O, 직관적 X
* QueryDSL -&gt; JPA지원 X 직관적 O
* JPA도 Native SQL을 지원하나 DBMS 변경시 SQL 작성 규칙이 다르면, 쿼리 모두 수정해야 되는 상황. 최대한 JPQL 사용.
* JPQL -&gt; 대량 데이터 수정 삭제하는 벌크연산 지원



**최초 작성일  2020.05.18**


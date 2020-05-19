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

### 10.2.1 기본 문법과 쿼리 API - JPQL 

```java
   TypedQuery<Meniber> query =
em. createQuery ("SELECT m FRCTd Member m" , Member. class) ;
List<Member> resultList = query.getResultList();
for (Member member : resultList) {
System.out.printin("member = ” + member);
```

### 10.2.2 파라미터 바인딩 - JPQL 

### 10.2.3  프로젝션 - JPQL 

### 10.2.4 페이징 API - JPQL 

### 10.2.5 집합과정렬 - JPQL 

### 10.2.6 JPQL 조인 - JPQL 

### 10.2.7 페치 조인 - JPQL 

### 10.2.8 경로표현식 - JPQL 

### 10.2.9 서브 쿼리 - JPQL 

### 10.2. 조건식 - JPQL 

### 10.2.11 다형성쿼리 - JPQL 

### 10.2.12 사용자 정의 함수 호출\(JPA 2.1\) - JPQL 

### 10.2.13 기타정리 - JPQL 

### 10.2.14 엔티티 직접 사용 - JPQL 

### 10.2.15 Named Query: 정적쿼리 - JPQL 



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

### 

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

## 정리 

1. JPQL은 SQL을 추상화해서 특정 DB에 종속되지 않는다.
2. Creteria나 QueryDQL은 JPQL을 만들어주는 비럳 역할을 할 뿐이므로 JPQL을 잘 알아야 한다.
3. Creteria나 QueryDQL 를 사용하면 동적으로 변하는 쿼리 작성 가능
4. Creteria  -&gt; JPA지원 O, 직관적 X QueryDSL -&gt; JPA지원 X 직관적 O
5. JPA도 Native SQL을 지원하나 DBMS 변경시 SQL 작성 규칙이 다르면, 쿼리 모두 수정해야 되는 상황. 최대한 JPQL 사용.
6. JPQL -&gt; 대량 데이터 수정 삭제하는 벌크연산 지원



**최초 작성일  2020.05.18**


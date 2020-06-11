---
description: 김영한 저 "자바 ORM 표준 JPA 프로그래밍"을 읽고 정리한 내용입니다.
---

# \#12 스프링 데이터 JPA

Repository들의 하는 일이 비슷한 경우가 있다. \(예를 들어, 메서드 - save, findAll 등\)   
이때는 **GenerericDAO**를 이용해 해결하는 방법이 있지만, **부모 클래스에 너무 종속**되고 구현 클래스 상속에 가지는 단점에 노출된다.

인터페이스만 작성하면 실행시점에, 스프링 데이터 JPA가 구현객체를 동적으로 생성해서 주입해준다.   
즉, 데이터 접근계층을 개발할 때 구현 클래스 없이 **인터페이스만 작성해도** 개발을 완료할 수 있다.

#### 구현 예시

```java
//개발자가 직접 구현체를 개발하지 않아도 된다.
public interface MemberRepository extends JpaRepository<Member, Long>{
    Member findByUsername(String username);
}

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

* JpaRepository -&gt;일반적인 CRUD 메소드 제공
* 위 findByUsername 메소드는 JpaRepository가 제공하지 않지만, **이름을 분석**해서 아래의 JPQL을 실행한다. \(쿼리 메소드 기능\)

```sql
select m from Member m where username =:username
```

스프링 프레임워크와 JPA를 사용하는 환경에서 스프링 데이터 JPA를 꼭 사용하자.

## 1. 설정 \(스프링 프레임워크 사용시\)

```java
// 1. pom.xml 종속성 설정 
<artifactld>spring-data-jpa</artifactld>

// 2. XML 설정
<jpa:repositories base-package="jpabook.jpashop.repository" />

// 3. 자바 설정 
@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {}
```

애플리케이션을 실행할 때 basePackage에 있는 리포지토리 인터페이스들을 찾아서, 해당 인터페이스를 구현한 클래스를 동적으로 생성한 다음 스프링 빈으로 등록한다. 

## 2. JpaRepository 주요 메소드

| **메소드**  | **설명** |
| :---: | :--- |
| **save\(S\)**  |  새로운 엔티티는 저장하고 이미 있는 엔티티는 수정한다. |
| **delete\(T\)**  | 내부에서 EntityManager.remove\(\)를 호출 |
| **findOne\(ID\)** | 내부에서 EntityManager.find\(\)를 호출 |
| **getOne\(ID\)**  | 엔티티를 프록시로 조회한다. 내부에서 EntityManager.getReferenc\(\)를 호출 |
|  **findAll\(...\)** | 정렬\(Sort\)이나 페이징\(Pageable\) 조건을 파라미터로 제공할 수 있다. |

## 3. 쿼리 메소드 기능 

1. 메소드 이름으로 쿼리 자동 생성  \(위 findByUsername\)
2. 메소드 이름으로 NamedQuery 호출 
3. @Query 사용해서 리포지토리 인터페이스에 쿼리 직접 정의

### 3.1 메소드 이름으로 쿼리 자동 생성

정해진 규칙에 따라서 메소드 이름을 지어야 한다.

| Keyword | Sample | JPQL snippet |
| :--- | :--- | :--- |
| And | findByLastname**And**Firstname | … where x.lastname = ?1 and x.firstname = ?2 |
| Or | findByLastname**Or**Firstname | … where x.lastname = ?1 or x.firstname = ?2 |
| Is,Equals | findByFirstname, findByFirstnameIs, findByFirstnameEquals | … where x.firstname = 1? |
| Between | findByStartDate**Between** | … where x.startDate between 1? and ?2 |
| LessThan | findByAge**LessThan** | … where x.age &lt; ?1 |
| LessThanEqual | findByAge**LessThanEqual** | … where x.age &lt;= ?1 |
| GreaterThan | findByAge**GreaterThan** | … where x.age &gt; ?1 |
| GreaterThanEqual | findByAge**GreaterThanEqual** | … where x.age &gt;= ?1 |
| After | findByStartDate**After** | … where x.startDate &gt; ?1 |
| Before | findByStartDate**Before** | … where x.startDate &lt; ?1 |
| IsNull | findByAgeI**sNull** | … where x.age is null |
| IsNotNull,NotNull | findByAge\(Is\)**NotNull** | … where x.age not null |
| Like | findByFirstname**Like** | … where x.firstname like ?1 |
| NotLike | findByFirstname**NotLike** | … where x.firstname not like ?1 |
| StartingWith | findByFirstname**StartingWith** | … where x.firstname like ?1 \(parameter bound with appended %\) |
| EndingWith | findByFirstname**EndingWith** | … where x.firstname like ?1 \(parameter bound with prepended %\) |
| Containing | findByFirstname**Containing** | … where x.firstname like ?1 \(parameter bound wrapped in %\) |
| OrderBy | findByAge**OrderBy**LastnameDesc | … where x.age = ?1 order by x.lastname desc |
| Not | findByLastname**Not** | … where x.lastname &lt;&gt; ?1 |
| In | findByAge**In** \(Collection&lt;Age&gt; ages\) | … where x.age in ?1 |
| NotIn | findByAge**NotIn** \(Collection&lt;Age&gt; age\) | … where x.age not in ?1 |
| True | findByActive**True**\(\) | … where x.active = true |
| False | findByActive**False**\(\) | … where x.active = false |
| IgnoreCase | findByFirstname**IgnoreCase** | … where UPPER\(x.firstame\) = UPPER\(?1\) |

엔티티의 필드명이 변경되면 인터페이스에 정의한 메소드 변경해야 한다.

### 3.2 NamedQuery 호출

```java
//어노테이션 사용시 
@Entity
@NamedQuery(
        name="Member.findByUsername",
        query="select m from Member m where m.username = :username")
public class Member {
        //...
}

//XML 사용시
<named-query name="Member.findByUsername">
        <query><CDATA[
                select m from Member m where m.username :username
        ]></query>
</named-query>

//사
public interface MemberRepository extends JpaRepository<Member, Long> {
        //이름 기반 파라미터 바인딩
        List<Member> findByUsername(@Param("username") String username); 
}
```

### 3.3 @Query, 리포지토리 메소드에 쿼리 직접 정의

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query("select m from Member m where m.username = ?1")
    Member findByUsername (String username);
}


// 네이티브 쿼리 사용시
@Query(value = "SELECT * FROM MEMBER WHERE USERNAME = ?0", nativeQuery = true)
// JPQL은 위치기반 1부터 네이티브쿼리는 0부터 시작한다. 
```

### 3.4 벌크성 수정 쿼리

**`@Modifying`**벌크성 수정 삭제시 사용 

```java
// 전
int bulkPriceUp(String stockAmount){
    String qlString =
        "update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";
    int resultcount = em.createQuery(qlString)
                        .setParameter("stockAmount", stockAmount)
                        .executeUpdate();
}

// 후
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount" )
int bulkPriceUp(@Param("stockAmount") String stockAmount);
```

### 3.5 반환타입 두종류 

```java
List<Member> findByName (String name); //컬렉션
Member findByEmail (String email); //단건
```

* 단건은 **`getSingleResult()`** 메소드를 호출
* 따러서, 두가지 예외 발생 가능 **`NonUniqueResultException`**  **`NoResultException`** 

### 3.6 페이징 처리 

* **`Sort`** **`Pageable`** 파라미터 
* 반환타입으로 Page를 사용하면, 페이징 기능을 제공하기 위해 검색된 전체   데이터 건수를 조회하는 **count 쿼리를 추가로 호출**한다.

```java
//count 쿼리 사용
Page<Member> findByName(String name, Pageable pageable);

//count 쿼리 사용 안 함
List<Member> findByName(String name, Pageable pageable);
List<Member> findByName (String name, Sort sort);
```

이름이 "김"으로 시작하는 사람을 찾아 첫번째\(0\) 페이지부터 10개씩 보여준다.

* **`Pageable`**  **`PageRequest`** **`getContent()`** **`getTotalPages()`** **`hasNextPage()`**

```java
public interface MemberRepository extends Repository<Member, Long> {
    // StartingWith 3.1 네이밍 규칙 따라서 생성 
    Page<Member> findByNameStartingWith(String name, Pageable Pageable);
}

// 실행 코드 
PageRequest pageRequest =
        new PageRequest(0, 10, new Sort(Direction.DESC, "name"));
Page<Member> result =
        memberRepository.findByNameStartingWith("김", pageRequest);
List<Member> members = result.getContent(); 
int totalPages = result.getTotalPages(); 
boolean hasNextPage = result.hasNextPage(); 
```

페이지 인터페이스

```java
public interface Page<T> extends Iterable<T> {
    int getNumber(); //현재 페이지
    int getSize(); "페이지 크기
    int getTotalPages(); //전체 페이지 수
    int getNumberOfElements(); //현재 페이지에 나올 데이터 수
    long getTotalElements(); //전체 데이터 수
    boolean hasPreviousPage(); //이전 페이지 여부
    boolean isFirstPagege(); //현재 페이지가 첫 페이지 인지 여부
    boolean hasNextPage(); //다음 페이지 여부
    boolean isLastPage(); //현재 페이지가 마지막 페이지 인지 여부
    Pageable nextPageable(); //다음 페이지 객체, 다음 페이지가 없으면 null
    Pageable previous Pageable (); //다음 페이지 객체, 이전 페이지가 없으면 null
    List<T> getContent(); //조회된 데이터
    boolean hasContent(); //조회된 데이터 존재 여부
    Sort getSort(); //정렬 정보
}
```

### 3.7 힌트 

**`QueryHints`** - JPA 구현체에게 제공하는 힌트

```java
@QueryHints(value = { QQueryHint(name = "org.hibernate.readonly",
                                value = "true")}, forCounting = true)
Page<Member> findByName(String name, Pagable pageable);
```

**`forCounting 속`** -값추가로 호출 하는 페이징을 위한 count 쿼리에도 쿼리 힌트를 적용할지를 설정하는 옵션\(\)디폴트 true \)

### 3.8 락

16장에서 자세히 배운다. 

**`LockModeType.PESSIMISTIC_WRITE`**

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByName(String name);
```

## 4. 명세 

책 Domain Driven Design는 SPECIFICATION 개념을 소개하는데, 스프링 데이터 JPA는 **JPA Criteria**로 이 개념을 사용할 수 있도록 지원한다. 명세를 이해하기 위해선 핵심단어 술어\(predicate\)를 알아야 한다.

명세기능을 사용하기 위해 리포지토리에서 **`JpaSpecificationExecutor`**를 상속받으면 된다.

```java
public interface OrderRepository 
    extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order> {
    //...
}
```

**`JpaSpecificationExecutor`**인터페이스 

```java
public interface JpaSpecificationExecutor<T> {
    T findOne (Specification<T> spec);
    List<T> findAll (Specification<T> spec);
    Page<T> findAll (Specification<T> spec, Pageable pageable);
    List<T> findAll (Specification<T> spec, Sort sort);
    long count (Specification<T> spec);
}
```

이름 명세와 주문상태 명세를 and로 조합해서 검색조건으로 사용한다.

```java
import static org.springframework.data.jpa.domain.Specifications.*;
import static jpabook.jpashop.domain.spec.OrderSpec.*;
public List<Order> findOrders (String name) {
    List<Order> result = orderRepository.findAll(
                        where(memberName(name)).and(isOrderStatus())
)；
return result;
```

Specifications는 명세들을 조립할 수 있도록 도와주는 클래스이다.   
\(where\(\), and\(\), or\(\), not\(\) 메소드를 제공\)

#### OrderSpec 명세 정의 코드 

```java
package jpabook.jpashop.domain;

import org.springframework.data.jpa.domain.Specification;
import org.springframework.util.StringUtils;
import javax.persistence.criteria.*;

public class OrderSpec {
    public static Specification<Order> memberName (final String
    memberName) {
        //편의상 익명 클래스 사용
        return new Specification<Order> () {
            public Predicate toPredicate(Root<Order> root,
                CriteriaQuery<?> query, CriteriaBuilder builder) {
                
                    if (StringUtils.isEmpty(memberName)) return null;
                    
                    Join<Order, Member> m = root.j oin("member", JoinType.INNER); 
                    
                    return builder.equal(m.get("name"), memberName);
            }
        }；
    }
    public static Specification<Order> isOrderStatus () {
        return new Specification<Order> () {
            public Predicate toPredicate(Root<Order> root,
                CriteriaQuery<?> query, CriteriaBuilder builder) {
                    return builder.equal(root.get("status"),
                        OrderStatus.ORDER);
                }
        }；
    }
}
```

toPredicate\(\) 메소드 구현하면 JPA Criteria의 Root, CriteriaQuery, CriteriaBuilder 클래스가 모두 파라미터로 주어진다. 이 파라미터들을 활용해서 적절한 검색 조건을 반환하면 된다. \(10장 Criteria부분 참고\)

## 5. 사용자 정의 리포지토리

스프링 데이터 JPA로 리포지토리를 개발하면 인터페이스만 정의하고 구현체는 만들지 않는다. 하지만 여 이유로 메소드를 직접 구현해야 할 때도 있다. 그렇다고 리포지토리를 직접 구현하면 공통 인터페이스가 제공하는 기능까지 모두 구현해야 한다. 스프링 데이터 JPA는 이런 문제를 우회해서 필요한 메소드만 구현할 수 있는 방법을 제공한다.

```java
public interface MemberRepositoryCustom {
    public List<Member> 社ndMemberCustom();
}
```

네이밍 규칙 - 리포지토리 인터페이스 이름 + Impl **`MemberRepositorylmpl`**

```java
public class MemberRepositorylmpl implements MemberRepositoryCustom {
    @0verride
    public List<Member> findMemberCustom() {
        //...
    }
}
```

MemberRepositoryCustom을 상속받는다. 

```java
public interface MemberRepository
    extends JpaRepository<Member, Long>, MemberRepositoryCustom { 
}
```

## 6. 웹 확장

* 스프링 MVC에서 사용할 수 있는 편리한 기능을 제공한다.
  * **도메인 클래스 컨버터 기능 -**  식별자로 도메인 클래스를 바로 바인딩
  * **페이징과 정렬 기능**
* **`SpringDataWebConfiguration`**을 스프링 빈으로 등록해야 된다.
* JavaConfig 사용시**`EnableSpringDataWebSupport`**어노테이션 사용

### 6.1 도메인 클래스 컨버터 기능

*  **`/member/memberUpdateForm?id=1`**
  * 식별자값이 1인 멤버 엔티티를 찾아 온다.

```java
// 컨버터 기능 사용전 
@Controller
public class MemberController {

    @Autowired MemberRepository memberRepository;
    
    @RequestMapping("member/memberUpdateForm")
    public String memberUpdateForm(@RequestParam("id") Long id,
        Model model) {
        Member member = memberRepository.findOne(id); // 이코드 필요없어 진다.
        model.addAttribute("member", member);
        return "member/memberSaveForm";
    }
}

// 명시적으로 아래를 써줄 필요가 없다. 
Member member = memberRepository.findOne(id);
```

참고로 도메인 클래스 컨버터를 통해 넘어온 회원 엔티티를 컨트롤러에서 직접 수정해도, 실제 데이터베이스에는 반영되지 않는다. 이것은 OSIV와 관련이 있는데, OSIV 사용 안할시 넘어온 엔티티가 준영속 상태에 있기 때문에 merge 하던가, OSIV를 사용하도록 설정해두면 자동으로 반영되게 된다. \(OSIV 13장 확\)

### 6.2 페이징과 정렬 기능

Pageable의 기본값은 page=0, size=20이다.

* **`/members?page=0&size=20&sort=name,desc&sort=address.city`**
  * 이름과 주소의 도시로 정렬하고 첫번째\(0\) 페이지부터 20개씩 보기.
  * 페이지 1부터 시작하고 싶으면 PageableHandlerMethodArgumentResolver 

```java
@RequestMapping(value = "/members", method = RequestMethod.GET)
public String list(Pageable pageable, Model model) {
    Page<Member> page = memberService.findMembers(pageable);
    model.addAttribute("members", page.getContent());
    return "members/memberList";
}
```

**`Pageable`**은 요청 파라미터\(page, size, sort\) 정보로 만들어진다.   
\(**`Pageable`**은 인터페이스고 실제는 **`PageRequest`** 객체가 생성된다\)

* **`/members?member_page=0&order_page=1`**
  * 페이징 정보가 둘 이상이면 접두사를 사용해서 구분한다.  **`@Qualifier`**

```java
public String list(
    @Qualifier("member") Pageable member Pageable,
    @Qualifier("order") Pageable order Pageable, ...){
    //...
}
```

## 7.  스프링 데이터 JPA가 사용하는 구현체 

공통 인터페이스는**`SimpleJpaRepository`** 클래스가 구현한다.

```java
@Repository
@Transactional(readonly = true)
public class SimpleJpaRepository<T, ID extends Serializable> 
    implements JpaRepository<T, ID>, JpaSpecificationExecutor<T> {

    @Transactional
    public <S extends T> S save(S entity) {
        // 새로운 엔티티면 persist, 있는 엔티티면 merge
        if (entityinformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
    
    //...
}
```

JPA의 모든 변경은 트랜잭션 안에서 이루어져 하기 때문에 @Transactional를 지정하였고, 이로써 서비스 계층에서 트랜잭션을 시작하지 않았어도 리포지토리에서 시작된다. readonly는 변경없는 트랜잭션에서 플러시를 생략해, 성능향상을 준다. 

Persistable 인터페이스를 구현하면 엔티티 존재 여부를 판단 로직을 추가할 수 있다.

```java
public interface Persistable<ID extends Serializable> extends Serializable {
    ID getld();
    boolean isNew();
}
```

## 참고 자료

{% embed url="https://docs.spring.io/spring-data/jpa/docs/1.5.0.RELEASE/reference/html/jpa.repositories.html" %}




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
    丄nt getNumberOfElements(); //현재 페이지에 나올 데이터 수
    long getTotalElements(); //전체 데이터 수
    boolean hasPreviousPage(); //이전 페이지 여부
    boolean isdddfsFirstPagege()z //현재 페이지가 첫 페이지 인지 여부
    boolean hasNextPage(); //다음 페이지 여부
    boolean isLastPage(); //현재 페이지가 마지막 페이지 인지 여부
    Pageable nextPageable(); //다음 페이지 객체, 다음 페이지가 없으면 null
    Pageable previous Pageable (); 卜음" 페이지 객체' 이전 페이지가 없으면 null
    List<T> getContent(); //조회된 데이터
    boolean hasContent(); //조회된 데이터 존재 여부
    Sort getSort();//정렬 정보
}
```



## 참고 자료

{% embed url="https://docs.spring.io/spring-data/jpa/docs/1.5.0.RELEASE/reference/html/jpa.repositories.html" %}






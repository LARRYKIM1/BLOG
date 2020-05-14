---
description: 본 위키는 자바 ORM표준 JPA 프로그래밍 책을 읽고 작성하였습니다.
---

# \[ \#04 Entity Mapping \]

## 엔티티 매핑 \(121페이지\)



JPA를 사용하는데 **가장 중요한 일은 엔티티와 테이블을 정확히 매핑**하는 것. 따라서 매핑 **어노테이션을 숙지**하고 사용해야 한다.

* JPA에서 크게 4가지 분류의 매핑 어노테이션 제공
  * 객체와 테이블 매핑 @Entity, @Table
  * 기본 키 매핑 @Id
  * 필드와 컬럼 매핑 @Column
  * 연관관계 매핑 @ManyToOne, @JoinColumn

### 4.1 @Entity \(123페이지\)

DB의 테이블과 매핑할 클래스를 JPA가 관리할 수 있게 지정해주는 어노테이션

* 속성

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| `name` | JPA에서 사용할 **엔티티 이름을 지정한다.** 보통 기본값인 클래스 이름을 사용한다. 만약 다른 패키지에 이름이 같은 엔티티 클래스가 있다면 이름을 지정해서 충돌하지 않도록 해야 한다. | 설정하지 않으면 클래스 이름을 그대로 사용한다.\(예, Member\) |

**주의사항**

* 기본 생성자는 필수\(파라미터가 없는 public 또는 protected 생성자\).   이유 : JPA가 엔티티 객체를 생성할 때 기본 생성자를 사용한다.
* final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
* 저장할 필드에 final을 사용하면 안 된다.

```java
@Entity(name = "Book")
public static class Book {

    @Id
    private Long id;

    private String title;

    private String author;

    //Getters and setters are omitted for brevity
}
```

### 4.2 @Table

엔티티와 매핑할 **테이블을 지정**한다.

* 속성

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| `name` | 매핑할 테이블 이름 | 엔티티 이름을 사용한다. |
| `catalog` | catalog 기능이 있는 데이터베이스에서 catalog를 매핑한다. |  |
| `schema` | schema 기능이 있는 데이터베이스에서 schema를 매핑한다. |  |
| `uniqueConstraints(DDL)` | DDL 생성 시에 유니크 제약조건을 만든다. 2개 이상의 복합 유니크 제약조건도 만들 수 있다. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용된다. 뒷부분 4.5 참고. |  |

> 언제 카탈로그와 스키마 속성 사용하는가? [설명](https://stackoverflow.com/questions/11184025/what-are-the-jpa-table-annotation-catalog-and-schema-variables-used-for)

카탈로그와 스키마는 DB 서버사이드쪽에서 정의하는 **네임스페이스**이다. DBMS 벤더들에 따라 카탈로그와 스키마 둘다 제공 또는 한쪽만 제공하는 차이가 있다. 예를 들어, 유저가 로그인을 할 때, 유저의 네임스페이스를 따라 조절해, 테이블내에 보여줄 데이터와 숨길 데이터를 지정할 수 있다.

* 스키마 - 형식적인 언어로 설명된 데이터베이스 구조이다.\(외부, 개념, 내부\)
* 카탈로그\(= meta-data\) - 시스템 내의 모든 객체에 대한 정의나 명세이다. \(기본 테이블, 뷰 테이블, 동의어\(synonym\)들, 인덱스들, 사용자들, 사용자 그룹 등등\)

```java
@Entity(name = "Book")
@Table(
    catalog = "public",
    schema = "store",
    name = "book"
)
public static class Book {

    @Id
    private Long id;

    private String title;

    private String author;

    //Getters and setters are omitted for brevity
}
```

### 4.3 다양한 매핑사용 \(124페이지\)

JPA 시작하기 장에서 개발하던 회원관리 프로그램에 **다음 요구사항이 추가**될 때 어떻게 회원 엔티티에 기능을 추가할까? 1. 회원은 일반 회원과 관리자로 구분해야 한다. 변수명 roleType 사용하기. 2. 회원 가입일과 수정일이 있어야 한다. 변수명 createdDate, lastModifiedDate 사용하기. 3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다. 변수명 description 사용하기.

* 변경전

```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    //Getter, Setter
    . . .
}
```

* 변경후

```java
@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING) // 2가지 속성 존재
    private RoleType roleType; //일반 회원과 관리자로 구분

    @Temporal(TemporalType.TIMESTAMP) // 3가지 속성 존재
    private Date createdDate;  // 회원 가입일

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;  // 수정일

    @Lob // 길이 제한이 없다.
    private String description; 

    //Getter, Setter
     . . .
}
```

```java
package jpabook.start;

public enum RoleType {
    ADMIN, USER
}
```

> 자바와 DB의 변수 타입이 다를텐데 어떻게 매칭이 될까?

### 4.4 데이터베이스 스키마 자동생성 \(125페이지\)

지금까지는 테이블을 먼저 생성하고 엔티티를 만들었지만 데이터베이스 스키마 자동생성 사용해서 엔티티를 만들고 **테이블은 자동생성 되도록 해보자.**

> 사용 이유 - 클래스의 매핑 정보를 보면 어떤 테이블에 어떤 컬럼을 사용하는지 알 수 있다.

* persistence.xml 수정-&gt;애플리케이션 실행 시점에 DB 테이블 자동 생성을 위하여 아래추가

  ```markup
  <property name="hibernate.hbm2ddl.auto" value="create">
  ```

* 콘솔에서 실행된 SQL문을 보기위해 설정

  ```markup
  <property name="hibernate.show_sql" value="true">
  ```

* 실행된 SQL 확인

```sql
Hibernate:
   drop table MEMBER if exists
Hibernate:
create table MEMBER (
   ID varchar(255) not null,
   NAME varchar(255),
   age integer,
   roleType varchar(255), // 회원 구분
   createdDate timestamp, // 가입일
   lastModifiedDate timestamp, // 수정일
   description clob, // 자기소개
   primary key (ID)
)
```

> 언제 스키마 자동 생성 사용? - 객체와 테이블을 매핑하는데 익숙하지 않을 때 사용

* hibernate.hbm2ddl.auto 속성

| 옵션 | 설명 |
| :--- | :--- |
| `create` | 기존 테이블을 **삭제하고 새로 생성한다.** DROP + CREATE |
| `create-drop` | create 속성에 추가로 **애플리케이션을 종료할 때 생성한 DDL을 제거한다.** DROP + CREATE + DROP |
| `update` | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 **변경 사항만 수정한다.** |
| `validate` | 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 **차이가 있으면** 경고를 남기고 **애플리케이션을 실행하지 않는다.** 이 설정은 DDL을 수정하지 않는다.\(update와 차이\) |
| `none` | 자동 생성 기능을 사용하지 않는다. |

**HBM2DDL 주의사항**

**운영 서버**에서 `create, create-drop, update`처럼 DLL을 수정하는 옵션은 **절대 사용하면 안 된다.** 오직 개발 서버나 개발 단계에서만 사용해야 한다. 이 옵션들은 **운영 중인 데이터베이스의 테이블이나 컬럼을 삭제할 수 있다.** 개발 환경에 따른 **추천 전략**은 다음과 같다.   
 • **개발 초기 단계**는 `create` 또는 `update`   
 • **초기화 상태로 자동화된 테스트**를 진행하는 개발자 환경과 CI 서버는 `create` 또는 `create-drop`   
 • **테스트 서버**는 `update` 또는 `validate`   
 • **스테이징과 운영 서버**는 `validate` 또는 `none`

**참고**

자바 언어는 관례상 roleType과 같이 **카멜\(Camel\) 표기법**을 주로 사용하고. 데이터베이스는 관례상 roleType과 같이 **언더스코어\(\_\)를** 주로 사용한다. 앞서 살펴본 Member 엔티티를 이렇게 매핑하려면 @Column의 name 속성을 명시적으로 사용해서 이름을 지어주어야 한다. 다음과 같이 매핑한다.

```java
@Column(name="role_type") 
String roleType;
```

이것이 자동화로 만들고 싶을시 persistence.xml에 naming\_strategy 추가하면 자동으로 엔티티명 보고 DB에 컬럼생성시 자동으로 언더스코어 표기법으로 매핑한다.

```markup
<property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy" />
```

### 4.5 DDL 생성기능 \(129페이지\)

* **회원이름이 필수 입력**되야 하고, **10자를 초과할 수 없다**는 제약조건을 추가해보자.
* 변경전

  ```java
    @Column(name = "NAME")
    private String username;
  ```

* 변경후

  ```java
    @Column(name = "NAME", nullable = false, length = 10) //추가 
    private String username;
  ```

* 유니크 제약조건 만들기

  ```java
  @Table(name="MEMBER", 
    uniqueConstraints = {@UniqueConstraint( 
        name = "NAME_AGE_UNIQUE", // 제약조건명
        columnNames = {"NAME", "AGE"})} // 제약조건 걸 컬럼명들(유일한 값)
       )
  public class Member {
    // 생략
  }
  ```

* 자동으로 유니크 제약조건이 생성된 쿼리 확인

  ```sql
  ALTER TABLE MEMBER ADD CONSTRAINT NAME_AGE_UNIQUE UNIQUE (NAME, AGE)
  ```

> `uniqueConstraints` 사용 이유 - 애플리케이션 개발자가 엔티티만 보고도 손쉽게 다양한 제약 조건을 파악 가능하다.

### 4.6 기본 키 매핑 \(131페이지\)

```java
@Entity
public class Member {

    @Id 
    @Column(name = "ID")
    private String id;
    . . .
}
```

지금까지 기본키를 @Id를 사용해서 애플리케이션에서 직접 할당했다. 기본 키를 애플리케이션에서 직접 할당하는 대신에 **데이터베이스\(MySQL - 오토 인크리멘트, Oracle - 시퀀스 오브젝트\)가 생성해주는 값을 사용하려면 어떻게 매핑해야 할까?**

#### 기본키 생성 전략 미리보기

* 직접할당 - 기본키를 애플리케이션에서 직접 할당한다.
* 자동생성
  * IDENTITY - 기본 키 생성을 데이터베이스에 위임한다.
  * SEQUENCE - 데이터베이스 시퀀스를 사용해서 기본 키를 할당한다.
  * TABLE  - 키 생성 테이블을 사용한다.
* 생성전략이 이렇게 다양한 이유는 데이터베이스 벤더마다 지원하는 방식이 다르기 때문이다. 예를들어, 오라클데이터베이스는 시퀀스를 제공하지만 MySQL은 시퀀스를 제공하지 않는다.
* 기본 키를 직접 할당하려면 @Id만 사용하면 되고, 자동 생성 전략을 사용하려면 @Id에 @GeneratedValue를 추가하고 원하는 키 생성 전략을 선택하면 된다.

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "ID")
    private String id;
    . . .
}
```

#### 4.6.1 기본키 직접할당 전략

기본 키를 직접 할당하려면 **@Id로 매핑**하면 된다. 기본 키 직접 할당 전략은 em.persist\(\)로 엔티티를 DB에 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법이다.

```java
   Board board = new Board();
   board.setId("id1"); //기본 키 직접 할당
   em.persist(board);
```

* @Id 적용 가능 자바 타입

  `자바 기본형`

  `자바 래퍼Wrapper형`

  `String`

  `java.util.Date`

  `java.sql.Date`

  `java.math.BigDecimal`

  `java.math.BigInteger`

#### 4.6.2 IDENTTITY 전략

기본 키 생성을 **데이터베이스에 위임하는 전략**이다. 주로 **MySQL, PostgreSQL, SQL Server, DB2에서 사용**한다. MySQL의 AUTO\_INCREMENT 기능은 데이터베이스가 기본 키를 자동으로 생성해준다.

```java
@Entity
public class Board {
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY) // DBMS에게 생성하도록 위임
   private Long id;
   . . .
}
```

```sql
CREATE TABLE BOARD (
   ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
   DATA VARCHAR(255)
)
```

```java
//IDENTITY 사용
private static void logic(EntityManager em) {
   Board board = new Board();
   em.persist(board);
   System.out.println("board.id = " + board.getId());
}
//출력: board.id = 1
```

**주의사항**

* 엔티티가 영속 상태가 되려면 **식별자가 반드시 필요**하다. 그런데 IDENTITY 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 em.persist\(\)를 호출하는 즉시 INSERT SQL이 데이터베이스에 전달된다. 따라서 이 전략은 트랜잭션을 지원하는 **쓰기 지연**이 동작하지 않는다.

#### 4.6.3 SEQUENCE 전략

데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트다. SEQUENCE 전략은 이 **시퀀스를 사용해서 기본 키를 생성한다.** 이 전략은 시퀀스를 지원하는 **오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용**할 수 있다.

```sql
CREATE TABLE BOARD (
   ID BIGINT NOT NULL PRIMARY KEY,
   DATA VARCHAR(255)
)

//시퀀스 생성
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```

```java
@Entity
@SequenceGenerator(
  name = "BOARD_SEQ_GENERATOR" // 필수
  sequenceName = "BOARD_SEQ", //매핑할 데이터베이스 시퀀스 이름
  initialvalue = 1, allocationsize = 1) //  allocationsize - 한 번 호출에 증가하는 수
public class Board {
   @Id @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "BOARD_SEQ_GENERATOR")
   private Long id;
   . . .
}
```

**@SequenceGenerator 속성 정리**

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| `name` | 식별자 생성기 이름 | 필수 |
| `sqeuenceName` | 데이터베이스에 등록되어 있는 시퀀스 이름 | hibernate\_sequence |
| `initialvalue` | DDL 생성 시에만 사용됨, 시권스 DDL을 생성할 때 처음 시작하는 수를 지정한다. | 1 |
| `allocationSize` | 시퀀스 한 번 호출에 증가하는 수（성능 최적화에 사용됨） | 50 |
| `catalog, schema` | 데이터베이스 catalog, schema 이름 |  |

> 왜 allocationSize 기본값을 50으로 해뒀지..?

**주의사항**

* SequenceGenerator.allocationSize의 기본값이 50\(50씩 증가\)인 것에 주의해야 한다. 기본값이 50인 이유는 최적화 때문이다.
* SEQUENCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다.
* 식별자를 구하려고 데이터베이스 시퀀스를 조회한다.
* 조회한 시퀀스를 기본 키 값으로 사용해 데이터베이스에 저장한다.
* 이 방법은 시퀀스 값을 선점하므로 여러 JVM이 동시에 동작해도 기본 키 값이 충돌하지 않는 장점이 있다. 반면에 데이터베이스에 직접 접근해서 데이터를 등록할 때 시퀀스 값이 한번에 많이 증가한다는 점을 염두해두어야 한다. 이런 상황이 부담스럽고 INSERT 성능이 중요하지 않으면 allocationSize의 값을 1로 설정하면 된다. **즉, 시퀀스에 접근하는 횟수를 줄이기 위해 50이 기본값인 것이다.**

#### 4.6.4 TABLE 전략

TABLE 전략은 **키 생성 전용 테이블**을 하나 만들고 여기에 **이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다.** 이 전략은 테이블을 사용하므로 **모든 데이터베이스에 적용할 수 있다.**

TABLE 전략을 사용하려면 키 생성 용도로 사용할 테이블을 만들어야 한다.

```sql
//  키 생성 용도로 사용할 테이블
create table MY_SEQUENCES (
   sequence_name varchar(255) not null , // 시퀀스 명
   next_val bigint, //
   primary key ( sequence_name )
)
```

```java
@Entity
@TableGenerator(
 name = "BOARD_SEQ_GENERATOR",
 table = "MY_SEQUENCES",
 pkColumnValue = "BOARD_SEQ", allocationsize = 1)
public class Board {
   @Id
   @GeneratedValue(strategy = GenerationType.TABLE,
   generator = "BOARD_SEQ_GENERATOR")
   private Long id;
   . . . 
}
```

```java
private static void logic(EntityManager em) {
   Board board = new Board();
   em.persist(board);
   System.out.printin("board.id = " + board.getId());
}
//출력: board.id = 1
```

**@TableGenerator 속성 정리**

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| `name` | 식별자 생성기 이름 | 필수 |
| `table` | 키생성 테이블명 | hibernate\_sequence |
| `pkColumnName` | 시퀀스 컬럼명 | sequence\_name |
| `valueColumnName` | 시퀀스 값 컬럼명 | next\_val |
| `pkColumnValue` | 키로 사용할 값 이름 | 엔티티 이름 |
| `initialvalue` | 초기 값, 마지막으로 생성된 값이 기준이다. | 0 |
| `allocationsize` | 시퀀스 한 번 호출에 증가하는 수（성능 최적화에 사용됨） | 50 |
| `catalog, schema` | 데이터베이스 catalog, schema 이름 |  |
| `uniqueConstraints(DDL)` | 유니크 제약 조건을 지정할 수 있다. |  |

**참고**

#### 4.6.5 AUTO 전략

데이터베이스의 종류도 많고 기본 키를 만드는 방법도 다양하다. GenerationType.AUTO는 선택한 데이터베이스 방언에 따라 **IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.** 예를 들어, **오라클을 선택하면 SEQUENCE를, MySQL을 선택하면 IDENTITY**를 사용한다.

```java
@Entity
public class Board {
   @Id
   @GeneratedValue(strategy = GenerationType.AUTO)
   private Long id;
}
```

* 기본값이 AUTO라 생략가능하다.
* **AUTO 전략의 장점**은 데이터베이스를 변경해도 코드를 수정할 필요가 없다는 것이다. 특히 키 생성 전략이 아직 확정되지 않은 개발 초기 단계나 프로토타입 개발 시 편리하게 사용할 수 있다. AUTO를 사용할 때 SEQUENCE나 TABLE 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 한다. 만약 스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본값을 사용해서 적절한 시퀀스나 키 생성용 테이블을 만들어줄 것이다.

#### 4.6 정리

* 직접 할당: em.persist\(\) 를 호출하기 전에 애플리케이션에서 **직접 식별자 값을 할당**해야 한다. 만약 식별자 값이 없으면 예외가 발생한다
* SEQUENCE: **데이터베이스 시퀀스에서 식별자 값을 획득**한 후 **영속성 컨텍스트에 저장**한다.
* TABLE: 데이터베이스 **시퀀스 생성용 테이블에서 식별자 값을 획득**한 후 **영속성 컨텍스트에 저장**한다.
* IDENTITY: **데이터베이스에 엔티티를 저장해서 식별자 값을 획득**한 후 **영속성 컨텍스트에 저장**한다\(IDENTITY 전략은 테이블에 데이터를 저장해야 식별자 값을 획득할 수 있다\)

### 4.7 필드와 컬럼 매핑 \(145페이지\)

사용할 일이 있을 때 찾아서 자세히 읽어보는 것을 권장.

> @Transient와 @Access만 집고 넘어가기.

| 분류 | 매핑 어노테이션 | 설명 |
| :--- | :--- | :--- |
| 필드와 컬럼 매핑 | @Column | 컬럼을 매핑한다. |
|  | @Enumerated | 자바의 enum 타입을 매핑한다. |
|  | @Temporal | 날짜 타입을 매핑한다. |
|  | @Lob | BLOB, CLOB 타입을 매핑한다. |
|  | @Transient | 특정 필드를 데이터베이스에 매핑하지 않는다. |
| 기타 | @Access | JPA가 엔티티에 접근하는 방식을 지정한다. |

#### 4.7.1 @Column

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| name | 필드와 매핑할 테이블의 컬럼 이름 | 객체의 필드 이름 |
| insertable | 엔티티 저장 시 이 필드도 같이 저장한다. false로 설정하면 이 필드는 데이터베이스에 **저장**하지 않는다. false 옵션은 읽기 전용일 때 사용한다. | true |
| updatable | 엔티티 수정 시 이 필드도 같이 수정한다. false로 설정하면 데이터베이스에 **수정**하지 않는다. false 옵션은 읽기 전용일 때 사용한다 | true |
| [table](http://wonwoo.ml/index.php/post/834) | 하나의 엔티티를 **두 개 이상의 테이블에 매핑할 때 사용**한다. 지정한 필드를 다른 테이블에 매핑할 수 있다. 자세한 사용법은 7.5절에서 다룬다. | 현재 클래스가 매핑된 테이블 |
| nullable\(DDL\) | null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다. | true |
| unique\(DDL\) | @Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다. 만약 두컬럼 이상을 사용해서 유니크 제약조건을 사용하려면 클래스 레벨에서 @Table.uniqueConstraints를 사용해야 한다 |  |
| columnDefinition\(DDL\) | 데이터베이스 컬럼 정보를 직접 줄 수 있다 | 필드의 자바타입과 방언정보를 사용해서 적절한 컬럼 타입을 생성한다. |
| length\(DDL\) | 문자 길이 제약조건, String 타입에만 사용한다 | 255 |
| precision, scale\(DDL\) | BigDecimal 타입에서 사용한다（Biginteger도 사용 할 수 있다 precision은 소수점을 포함한 전체 자릿수를. scale은 소수의 자릿수다. 참고로 double, float 타입에는 적용되지 않는다. 아주 큰 숫자나 정밀한 소수를 다루어야 할 때만 사용한다. | precision = 19, scale=2 |

#### 4.7.2 @Enumerated

자바의 enum 타입을 매핑할 때 사용한다.

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| value | • EnumType.ORDINAL：enum 순서를 데이터베이스에 저장   • EnumType.STRING：enum 이름을 데이터베이스에 저장 | EnumType.ORDINAL |

#### 4.7.3 @Temporal

날짜 타입\(java.util.Date, java.util.Calendar\)을 매핑할 때 사용한다.

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| value | • TemporalType.DATE: 날짜, 데이터베이스 date 타입과 매핑（예: 2013-10-11）  • TemporalType.TIME：시간, 데이터베이스 time 타입과 매핑（예: 11:11:11）  • TemporalType.TIMESTAMP：날짜와 시간, 데이터베이스 timestamp 타입과 매핑（예: 2013-10-11 11:11:11） | TemporalType은 필수로 지정해야 한다. |

```java
   @Temporal(TemporalType.DATE)
   private Date date; //날짜

   @Temporal(TemporalType.TIME)
   private Date time; //시간

   @Temporal(TemporalType.TIMESTAMP)
   private Date timestamp; //날짜와시간

   //== 생성된 DDL==//
   date date,
   time time,
   timestamp timestamp,
```

방언 덕분에 자바 변수 타입에 의해 생성되는 타입은 다음과 같다.

* datetime: MySQL
* timestamp: H2, 오라클, PostgreSQL

#### 4.7.4 @Lob

데이터베이스 BLOB, CLOB 타입과 매핑한다. @Lob는 지정할 수 있는 속성이 없다. 대신에 매핑하는 **필드 타입이 문자면 CLOB으로 매핑**하고 **나머지는 BLOB으로 매핑**한다.

```java
CLOB: String, char [], java.sql.CLOB
BLOB: byte [], java.sql.BLOB
```

#### 4.7.5 @Transient

이 필드는 매핑하지 않는다. 따라서 데이터베이스에 저장하지 않고 조회하지도 않는다. 객체에 임시로 어떤 값을 보관하고 싶을 때 사용한다.

> 언제 사용하는 것인가? 아래참고

#### 4.7.6 @Access

JPA가 엔티티 데이터에 접근하는 방식을 지정한다.

* 필드 접근: AccessType.FIELD로 지정한다. 필드에 직접 접근한다. 필드 접근 권한이 private이어도 접근할 수 있다. 
* 프로퍼티 접근: AccessType.PROPERTY로 지정한다. 접근자를 사용한다. 

@Access를 설정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다. @Id가 필드에 있으므로 @Access\(AccessType.FIELD\)로 설정한 것과 같다. 따라서 @Access는 생략해도 된다.

* 프로퍼티 접근

  \`\`\`java @Entity public class Member { @Id private String id;

  @Transient private String firstName;

  @Transient private String lastName;

@Access\(AccessType.PROPERTY\) //프로퍼티 접근 방식을 사용 public String getFullName\(\) { return firstName + lastName; // 엔티티를 저장하면 회원 테이블의 FULLNAME 컬럼에 firstName + lastName의 결과가 저장 } . . . }

```text
<br/>

## 4.8 정리
 이 장을 통해 **객체와 테이블 매핑, 기본 키 매핑, 필드와 컬럼 매핑**에 대해 알아보았다. 그리고 데이터베이스 **스키마 자동 생성하기 기능**도 알아보았는데, 이 기능을 사용하면 엔티티 객체를 먼저 만들고 테이블은 자동으로 생성할 수 있다. JPA는 다양한 기본 키 매핑 전략을 지원한다. 기본 키를 애플리케이션에서 직접 할당하는 방법부터 데이터베이스가 제공하는 기본 키를 사용하는 SEQUENCE,
IDENTITY, TABLE 전략에 대해서도 알아보았다. **이 장에서 다룬 회원 엔티티는 다른 엔티티와 관계가 없다. 회원이 특정 팀에 소속해야 한다면 어떻게 해야 할까?** 다음 장을 통해 연관관계가 있는 엔티티들을 어떻게 매핑하는지 알아보자.

<br/>

## [실전 예제] 요구사항 분석과 기본 매핑 (154페이지)

### 실전 예제 구현 목표

+ 회원 기능
  + 회원 등록
  + 회원 조회
+ 상품 기능
  + 상품 등록
  + 상품 수정
  + 상품 조회
+ 주문 기능
  + 상품 주문
  + 주문 내역 조회
  + 주문 취소

<br/> 

### [실전 예제] 1.1 요구사항 분석 
+ 회원은 상품을 주문할 수 있다.
+ 주문시 여러 종류의 상품을 선택할 수 있다.

<br/> 

### [실전 예제] 1.2 도메인 모델 분석
요구사항을 분석하여 **엔티티를 예측**해보자.

![](https://bit.ly/2xyrqIi)

- **회원과 주문의 관계**: 회원은 여러 번 주문할 수 있으므로 회원과 주문은 일대다 관계다. 
- **주문과 상품의 관계**: 주문할 때 여러 상품을 함께 선택할 수 있고, 같은 상품도 여러번 주문될 수 있으므로 둘은 다대다 관계다. 하지만 이런 **다대다 관계**는 관계형 데이터베이스는 물론이고 엔티티에서도 **거의 사용하지 않는다.** 따라서 **주문상품이라는 연결 엔티티를 추가**해서 다대다 관계를 일대다, 다대일 관계로 풀어냈다. 그리고 주문상품에는 해당 상품을 구매한 금액과 수량 정보가 포함되어 있다.

<br/> 

### [실전 예제] 1.3 테이블 설계
요구사항을 분석해서 **데이터베이스 테이블을 만들자.**

![](https://bit.ly/2wMR6QO)
- **회원(MEMBER)**: 이름(NAME)과 주소 정보를 가진다. 주소는 CITY, STREET, ZIPCODE로 표현한다.
- **주문(ORDERS)**: 상품을 주문한 회원(MEMBER_ID)을 외래 키로 가진다. 그리고 주문날짜(ORDERDATE)와 주문 상태(STATUS)를 가진다. 주문 상태는 주문(ORDER)과 취소(CANCEL)를 표현할 수 있다.
- **주문상품(ORDER_ITEM)**: 주문(ORDER_ID)과 주문한 상품(ITEM_ID)을 외래 키로 가진다. 주문 금액(ORDERPRICE)，주문 수량(COUNT) 정보를 가진다.
- **상품(ITEM)**: 이름(NAME), 가격 (PRICE), 재고수량(STOCKQUANTITY)을 가진다. 상품을
주문하면 재고수량이 줄어든다.


|회원|주문|주문상품|상품|
|-|-|-|-|
|홍길동|주문01|2개|연필|
|||3개|지우개|
|||1개|필통|
||주문02|2개|필통|
|||4개|연필깍이|
|||2개|깔창|

<br/> 

### [실전 예제] 1.4 엔티티 설계와 매핑
**설계한 테이블을 기반**으로 엔티티를 만들어보자.

![](https://bit.ly/3br6RME)

#### 회원 엔티티 코드
```java
@Entity
public class Member {

   @Id @GeneratedValue
   @Column(name = "MEMBER_ID")
   private Long id;

   private String name;

   private String city;
   private String street;
   private String zipcode;

   //Getter, Setter
   . . . 
}
```

**회원 엔티티 설명**

회원은 이름\(name\)과 주소 정보를 가진다. 주소는 city, street, zipcode로 표현한다. 식별자는 @Id와 @GeneratedValue를 사용해서 데이터베이스에서 자동 생성되도록 했다. **@GeneratedValue의 기본 생성 전략은 AUTO이므로 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE중 하나가 선택된다.** 예제에서는 H2 데이터베이스를 사용하는데 이 데이터베이스는 SEQUENCE를 사용한다. 다른 엔티티들에 대해서도 같은 키 생성 전략을 사용하겠다.

**주문 엔티티 코드**

```java
@Entity
@Table(name = "ORDERS")
public class Order {

   @Id @GeneratedValue
   @Column (name = "ORDER_ID")
   private Long id;

   @Column(name = "MEMBER_ID")
   private Long memberId;

   // 년월일 시분초 모두 사용하므로 TIMESTAMP 속성을 사용
   @Temporal(TemporalType.TIMESTAMP) // 생략해도 JPA가 TIMESTAMP로 디폴트 설정
   private Date orderDate; //주문날짜

   @Enumerated(EnumType.STRING) // 열거형 사용
   private OrderStatus status;//주문상태

   //Getter, Setter
   . . . 
}
```

```java
public enum OrderStatus {
 ORDER, CANCEL // 주문과 취소
}
```

**주문 엔티티 설명**

주문은 상품을 주문한 회원\(memberId\)의 외래키 값과 주문 날짜\(orderDate\), 주문 상태\(status\)를 가진다.

**주문상품 엔티티 코드**

```java
@Entity
@Table(name = "ORDER_ITEM")
public class OrderItem {

   @Id @GeneratedValue
   @Column(name = "ORDER_ITEM_ID")
   private Long id;

   @Column (name = "ITEM_ID")
   private Long itemId;

   @Column(name = "ORDER_ID")
   private Long orderId;

   private int orderPrice; //주문가격
   private int count; //주문수량

   //Getter, Setter
   . . .
}
```

**주문상품 엔티티 설명**

주문상품은 주문\(orderId\)의 외래 키 값과 주문한 상품\(itemId\)의 외래 키 값을 가진다. 그리고 주문 금액\(orderPrice\)과 주문수량\(count\) 정보를 가진다.

**상품 엔티티 코드**

```java
@Entity
public class Item {

   @Id @GeneratedValue
   @Column(name = "ITEM_ID")
   private Long id;

   private String name; //이름
   private int price; //가격
   private int stockQuantity; //재고수량

   //Getter, Setter
}
```

**상품 엔티티 설명**

상품은 이름\(name\), 가격\(price\), 재고수량\(stockQuantity\) 정보를 가진다.

#### \[실전 예제\] 1.5 데이터 중심 설계의 문제점

이 예제의 엔티티 **설계가 이상하다는 생각**이 들었다면 **객체지향 설계를 의식하는 개발자**고, 그렇지 않고 자연스러웠다면 데이터 중심의 개발자일 것이다. 객체지향설계는 각각의 객체가 맡은 역할과 책임이 있고 관련 있는 **객체끼리 참조하도록 설계**해야 한다. 지금 **이 방식은 객체 설계를 테이블 설계에 맞춘 방법**이다. 특히 테이블의 외래키를 객체에 그대로 가져온 부분이 문제다. 왜냐하면 관계형 데이터베이스는 연관된 객체를 찾을 때 외래 키를 사용해서 조인하면 되지만 **객체에는 조인이라는 기능이 없다.** 객체는 연관된 객체를 찾을 때 **참조를 사용해야 한다.** 설계한 엔티티로 데이터베이스 스키마 자동 생성하기를 실행해보면 ERD에 나온대로 테이블이 생성된다. 하지만 객체에서 참조 대신에 데이터베이스의 외래키를 그대로 가지고 있으므로 order.getMember\(\) 처럼 **객체 그래프를 탐색할 수 없고** 객체의 특성도 살릴 수 없다. 그리고 객체가 다른 객체를 참조하지도 않으므로 UML도 잘못되었다. **결국, 객체는 외래키 대신에 참조를 사용해야 한다.**

이렇게 **외래키만 가지고 있으면** 연관된 엔티티를 찾을 때 **외래키로 데이터베이스를 다시 조회해야 한다.** 예를 들어, 주문을 조회한 다음 주문과 연관된 회원을 조회하려면 다음처럼 외래키를 사용해서 다시 조회해야 한다.

```java
Order order = em.find(Order.class, orderId);
```

```java
//외래키로 다시 조회
Member member = em.find(Member.class, order.getMemberId());
```

객체는 참조를 사용해서 연관관계를 조회할 수 있다. 따라서 **다음처럼 참조를 사용하는 것이 객체지향적인 방법**이다.

```java
Order order = em.find(Order.class, orderId); 
Member member = order.getMember(); //참조 사용
```

실전예제를 정리하자면, 객체는 참조를 사용해서 연관된 객체를 찾고 테이블은 외래 키를 사용해서 연관된 테이블을 찾으므로 둘 사이 에는 큰 차이가 있다. JPA는 객체의 참조와 테이블의 외래 키를 매핑해서 객체에서는 참조를 사용하고 테이블에서는 외래 키를 사용할 수 있도록 한다. **다음 장을 통해 참조와 외래 키를 어떻게 매핑하는지 알아보자.**


---
description: 김영한 저 "자바 ORM 표준 JPA 프로그래밍"을 읽고 정리한 내용입니다.
---

# \#03 영속성 관리

**영속성 컨택스트 =** **JPA가 애플리케이션서버와 DB서버 사이에서 엔티티를 관리하는 공간.**

## 영속성 상태 4가

1. 비영속\(**new/transient**\) : 영속성 컨텍스트와 전혀 관계가 없는 상태
2. 영속\(**managed**\): 영속성 컨텍스트에 저장된 상태
3. 준영속\(**detached**\): 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제\(**removed**\): 삭제된 상태

## 생명주기

![&#xAE40;&#xC601;&#xD55C;, &#x300C;&#xC790;&#xBC14; ORM &#xD45C;&#xC900; JPA &#xD504;&#xB85C;&#xADF8;&#xB798;&#xBC0D;&#x300D;, &#xC5D0;&#xC774;&#xCF58;, 2015](../../.gitbook/assets/image%20%2816%29.png)

**비영속 - 객체만 생성하고 아직 영속성컨택스트에 저장하지 않음.**

```java
Member member = new Member();
member.setId("member1);
member.setUserName("회원1");
```

**영속 - 영속성 컨텍스트 관리하는 상**

```java
em.persist(member);
```

**준영속 - 엔티티를 영속성 컨텍스트가 더이상 관리하지 않으면 준영속 상태**

```java
em.detach() // 특정 엔티티만 준영속 상태로 만들 수 있다.
em.close() // 영속성 컨텍스트를 닫는다.
em.clear() // 영속성 컨텍스트의 데이터를 비운다.
```

**삭제 - 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제** 

```java
em.remove(member);
```

## 영속성 컨텍스트는의 특징

1. 엔티티를 **식별자** 값으로 구분 \(식별자값은 **데이터베이스 기본 키와 매핑**\)
2.  **flush**로 데이터베이스에 반영 \(커밋해도 플러시 포\)
3. 다양한 장점 \(중\)
   * **1차캐시** 
   * **동일성 보장**
   * **트랜잭션을 지원하는 쓰기 지연**
   * **변경 감지**
   * **지연 로딩**

### 다양한 장점 맛보기

#### **1차캐시** 

* commit이나 flush 할 때까지 데이터베이스에 저장 X
* 1차 캐시의 키는 식별자의 값
* 티티를 조회할 때 1차 캐시 먼저

#### **동일성 보장**

* 메모리 낭비 방지

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

a == b; //true
```

#### 트랜잭션을 지원하는 **쓰기 지연**

* 트랜잭션을 커밋하기 직전까지 ****내부 쿼리 저장소에 SQL을 차곡차곡 저

#### **변경 감지**

* 관리하는 영속 상태의 엔티티에만 적용
* 따로 UPDATE SQL 없이 엔티티를 find\(\)해서 setter로 수정만 해줘도 DB 반영
  * 플러시 시점에 스냅샷\(저장된 엔티티 최초상태\)과 엔티티를 비교해서 DB에 반영

#### **지연 로딩**

* 자신과 연관된 엔티티를 실제로 사용 시점 연관된 엔티티를 조회

### 코드 전체 분석전 부분 분석

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistence.xml 내 유닛명");
```

**하나의 데이터베이스는 하나의 엔티티 매니저 팩토리**\(EntityManagerFactory\)를 가진다.

```java
EntityManager em = emf.createEntityManager();
```

**엔티티 매니저**는 여러 **스레드가 동시에 접근해도 안전**하므로 서로 다른 스레드 간에 공유해도 된다.  
**트랜잭션을 시작시 데이터베이스 연결**하여 **커넥션을 획득**한다.

## 전코드 분석

#### &lt; JpaMain.class &gt;

```java
import javax.persistence.*;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {
    
            //엔티티 매니저 팩토리 생성
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
            EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
    
            EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
    
            try {
    
         // ★★ 트랜잭션을 시작시 데이터베이스 연결하여 커넥션을 획득한다. ★★ 
                tx.begin(); //트랜잭션 시작
                logic(em);  //비즈니스 로직
                tx.commit();//트랜잭션 커밋
    
            } catch (Exception e) {
                e.printStackTrace();
                tx.rollback(); //트랜잭션 롤백
            } finally {
                em.close(); //엔티티 매니저 종료
            }
    
            emf.close(); //엔티티 매니저 팩토리 종료
        }
    
        public static void logic(EntityManager em) {
    
            String id = "id1";
            Member member = new Member();
            member.setId(id);
            member.setUsername("지한");
            member.setAge(2);
    
            //등록
            em.persist(member);
    
            //수정
            member.setAge(20);
    
            //한 건 조회
            Member findMember = em.find(Member.class, id);
            System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());
    
            //목록 조회
            List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
            System.out.println("members.size=" + members.size());
    
            //삭제
            em.remove(member);
    
        }
    }
}
```

#### &lt; Member.class &gt;

```java
@Entity
@Table(name="MEMBER") //MEMBER 테이블에 매
public class Member {  
    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    private Integer age;
    
    // GETTER, SETTER OMITTED
}
```

#### &lt; persistence.xml &gt;

```markup
<persistence-unit name="jpabook"> <!-- 유닛명 엔티티 매니저 팩토리에 적어 -->
    <properties>
            <!-- H2 데이터베이스 드라이버 및 설정 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/> <!-- 기본 ID -->
            <property name="javax.persistence.jdbc.password" value=""/> <!-- 기본 비번 없음 -->
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>

            <!-- 방언 사용 -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 테이블 자동 생성 -->
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            <!-- 실행되는 SQL 을 보여줌 -->
            <property name="hibernate.show_sql" value="true" />
            <!-- 실행되는 SQL을 이쁘게 -->
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.id.new_generator_mappings" value="true"/>
    </properties>
</persistence-unit>
```

#### 콘솔로 본 내부과정 정

1. setter\(\)들 \(아직 INSERT가 날아가지 않는다.\)   
2. persist\( \) 데이터베이스에 바로 저장하지 않고 @Id 값으로 영속성 컨택스트 내에서 유일한 식별자값을 만든다. \(아직 INSERT가 날아가지 않는다.\)   
3. setAge\(20\) - 업데이트문이 날아가지 않는다.   
4. em.find\(Member.class, id\) - 1차캐시에서 찾아온다. 아직도 데이터베이스까지의 통신이 없다.  
5. em.createQuery\("select m from Member m", Member.class\).getResultList\(\);   
      리스트 조회시 콘솔을 보니 insert -&gt; update -&gt; select 문이 차례대로 날아갔다.궁

> **궁금? -** 왜 insert -&gt; update -&gt; select로 할까 1차캐시에서 나이를 20으로 변경했으면 insert -&gt; select 만 해도 되는 것 안닌가?


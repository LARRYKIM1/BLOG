# \#16 트랜잭션과 락, 2차캐시

## 1. 트랜잭션과 락 

트랜잭션 간에 격리성을 완벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 한다. 이렇게 하면 동시성 처리 성능이 매우 나빠진다.

* 트랜잭션의 격리 수준을 4단계 \(ANSI 표준\)
  * READ UNCOMMITED（커밋되지 않은 읽기）
  * READ COMMITTED（커밋된 읽기）
  * REPEATABLE READ（반복 가능한 읽기）
  * SERIALIZABLE （직렬화가능）

아래로 갈수록 격리수준이 높아진다 = 동시성이 줄어든다는 말.

* 격리수준 문제점 
  * DIRTY READ \(READ UNCOMMITED에서 발생\)
  * NON-REPEATABLE READ \(READ COMMITED에서 발생\)

    예, 트랜잭션1에서 조회후 트랜잭션1에서 다시 읽었는데 다른 데이터값. 트랜잭션2에서 그 사이 변경해버렸다.

  * PHANTOM READ \(REPEATABLE READ에서 발생\)

    예, 트랜잭션1에서 5개 데이터 나왔는데 다시 조회하니 6개. 트랜잭션2에서 그사이 추가해버렸다.

    위에는 다른 트랜잭션에서 수정시 발생했고 이번에는 추가시 발생.

참고, 최근 DB에서 동시성 처리를 위해 락보다 MVCC를 이용한다.

* 두번의 갱신 분실 문제\(Second lost updates problem\) 해결법 
  * 마지막 커밋만 인정하기: 마지막에 커밋한 사용자의 내용만 인정한다. 
  * 최초 커밋만 인정하기: 사용자 A가 이미 수정을 완료했으므로 사용자 B가 수정을 완료할 때 오류가 발생한다. 
  * 충돌하는 갱신 내용 병합하기: 양쪽 수정사항을 병합한다.

기본적으로 첫번째가 많이 사용되고, 상황에 따라 두번째가 합리적일 수 있다.

### @Version

* 낙관적 락을 사용하기 위한 어노테이션
* 적용가능 타입
  *  Long \(long\)     / Integer \(int\)     / Short \(short\)     / Timestamp
* 엔티티를 수정할 때 마다 버전  이 하나씩 자동으로 증가
  * 수정할 때, 조회 시점의 버전과 수    정 시점의 버전이 다르면 예외가 발생한다.
  * **`OptimisticLockException`**
    * 스프링 예외 추상화 **`ObjectOptimisticLockingFailureException`**
* 최초 커밋만 인정하기가 적용된다.
  * 즉, 커밋하는 시점에 충돌을 알수 있다.
* 옵션 OPTIMISTIC을 추가  하면 엔티티를 조회만 해도 버전을 체크한다.
* 임베디드 타입과 값 타입 컬렉션은 논리적인 개념상 해당 엔티티의 값이므로, 수정하면 엔티티의 버전 이 증가한다. 멤버:주소 -&gt; 주소 변경시, 멤버 버전 증가.
* 벌크 연산은 버전을 무시한다. 딸라서 버전을 증가하려면 버전 필드를 강제로 증가시켜야 한다.

```sql
// WHERESET에 버전 조건을 건다. 동일하지 않으면 예외발생.
UPDATE BOARD
SET
    TITLE=?,
    VERSION=? (버전 + 1 증가)
WHERE
    ID=?
    AND VERSION=? (버전 비교) 
```

### JPA 락 사용

* JPA 사용시 추천 전략 
  * READ COMMITTED + 낙관적 버전 관리 
  * 어느정도 동시성을 가져가며, 두번의 갱신 내역 분실 문제 예방 가능하다.
* 락 적용 가능 위치 
  * EntityManager.lock\(\), EntityManger.find\(\), EntityManager.refresh\(\) 
  * Query.setLockMode\(\) \(TypeQuery 포함\) 
  * @NamedQuery

```java
//사용 예
Board board = em.find(Board.class, id, LockModeType.OPTIMISTIC);

//이렇게도 가능
Board board = em.find(Board.class, id);
em.lock(board, LockModeType.OPTIMISTIC);
```

* LockModeType 종류 

  * OPTIMISTIC / OPTIMISTIC\_FORCE\_INCREMENT 
  * PESSIMISTIC\_WRITE / PESSIMISTIC\_READ / PESSIMISTIC\_FORCE\_INCREMENT 
  * NONE / READ / WRITE

* 낙관적 락은 트랜잭션을 **커밋하는 시점에 충돌을 알수 있다. ``**
* 락 옵션 없이 @Version만 있어도 낙관적 락이 적용된다.





## 2. 2차 캐시

일반적인 웹 애플리케이션 환경은 트랜잭션을 시작하고 종료할 때까지만 1차 캐시가 유효하다. OSIV를 사용해도 클라이언트의 요청이 들어올 때부터 끝날 때까지만 1차 캐시가 유효하다. 따라서 애플리케이션 전체로 보면 데이터베이스 접근 횟수를 획기적으로 줄이지는 못한다.

### 1차캐시 

1차 캐시는 영속성 컨텍스트 내부에 있다. 트랜잭션을 커밋하거나 플러시를 호출하면, 1차 캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화 한다.

### 2차캐시  \(shared cache, second level cache\)

* 2차 캐시는 애플리케이션 범위의 캐  시여서 애플리케이션을 종료할 때까지 캐시가 유지된다.
* 2차 캐시는 동시성을 극대화하려고 캐시한 객체를 직접 반환하지 않고 **복사본을 만들어서 반환**한다. 캐시한 객체를 그대로 반환하면 여러 곳에서 같은 객체를 동시에 수정하는 문제가 발생할 수 있다. 이 문제를 해결하려면 객체에 락을 걸어야 하는데 이렇게 하면 동시성이 떨어질 수 있다. **락에 비하면** 객체를 복사하는 비  용은 **아주 저렴하다.**

![&#xAE40;&#xC601;&#xD55C;, &#x300C;&#xC790;&#xBC14; ORM &#xD45C;&#xC900; JPA &#xD504;&#xB85C;&#xADF8;&#xB798;&#xBC0D;&#x300D;, &#xC5D0;&#xC774;&#xCF58;, 2015](../../.gitbook/assets/image%20%2857%29.png)

#### 2차 캐시 사용하기 위한 설정 

**`@Cacheable`** 기본값 true

```java
@Cacheable
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    //...
}
```

```markup
persistence.xml 사용시
<persistence-unit name="test">
    <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
</persistence-unit>

스프링 프레임워크 사용시 
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="sharedCacheMode" value="ENABLE_SELECTIVE" />
...
```

* shared-cache-mode 속성
  * ALL / NONE / ENABLE\_SELECTIVE / DISABLE\_SELECTIVE / UNSPECIFIED








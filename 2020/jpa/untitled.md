---
description: '김영한 저 "자바 ORM 표준 JPA 프로그래밍, 2015"을 읽고 정리한 내용입니다.'
---

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

### 1.1 @Version

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
* 벌크 연산은 버전을 무시한다. 따라서 버전을 증가하려면 버전 필드를 강제로 증가시켜야 한다.

```sql
// WHERE에 버전 조건을 건다. 동일하지 않으면 예외발생.
UPDATE BOARD
SET
    TITLE=?,
    VERSION=? (버전 + 1 증가)
WHERE
    ID=?
    AND VERSION=? (버전 비교) 
```

### 1.2 JPA 락 사용

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

#### 1.2.1 낙관적 락 

발생가능 예외  
**`OptimisticLockException` `StaleObjectStateException`** **`ObjectOptimisticLockingFailureException`**

* OPTIMISTIC 
  * 엔티티를 조회만 해도 버전을 확한다.
  * 한 번 조회한 엔티티는 트랜잭션을 종료할 때까지 다른 트랜잭션에서 변경하지 않음을 보장한다.
  * 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알수 있다.

```java
Board board = em.find(Board.class, id, LockModeType.OPTIMISTIC);

//중간에 다른 트랜잭션에서 동일 게시물을 수정해서 버전 증가 발생

//커밋 시점에 버전 정보 검증, 예외 발생
//(데이터베이스 version=2, 엔티티 version=1)
tx.commit();
```

* OPTIMISTIC\_FORCE\_INCREMENT 
  * 엔티티를 수정하지 않아도 트랜잭션을 커밋할 때 UPDATE 쿼리를 사용해서 버전 정보를 강제로 증가시킨다. 이때 데이터베이스의 버전이 엔티티의 버전과 다르면 예외가 발생한다. 
  * 추가로 엔티티를 수정하면 수정시 버전 UPDATE 가 발생한다. 따라서 총 2번의 버전 증가가 나타날 수 있다.
  * 예, Aggregate Root는 수정하지 않았지만 Aggregate Root가 관리하는 엔티티를 수정했을 때 Aggregate Root의 버전을 강제로 증가시킬 수 있다.

#### 1.2.2 비관적 락 

발생가능 예외  
**`PessimisticLockingFailureException (스프링 추상화 예외)`** **`PessimisticLockException`**

* PESSIMISTIC\_WRITE 
  * 비관적 락이라 하면 일반적으로 이 옵션을 뜻한다.
  * 쓰기 락을 걸 때 사용
  * NON-REPEATABLE READ를 방지한다. 즉, 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.
* PESSIMISTIC\_READ 
  * 반복 읽기만 하고 수정하지 않는 용도로 락을 걸 때 사용한다.
*  PESSIMISTIC\_FORCE\_INCREMENT
  * 비관적 락중 유일하게 버전 정보를 사용한다

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

**캐시 조회 저장 방식 설정**

* 캐시를 무시하고 데이터베이스를 직접 조회하거나 캐시를 갱신하려면 캐시 조회 모드와 캐시 보관 모드를 사용
* 캐시 조회 모드
  * retrieveMode\(프로퍼티\), CacheRetrieveMode\(설정 옵션\)
* 캐시 보관 모드
  * storeMode\(프로퍼티\), CacheStoreMode\(설정 옵션\)
* 캐시 모드 
  * 1. em.setProperty\(\) - 엔티티 매니저 단위로 설정
  * 2. em.find\(\), em.refresh\(\)에 설정
  * 3. Query.setHint\(\) 사용

```java
//캐시 조회 모드
public enum CacheRetrieveMode {
  USE,    // 캐시에서 조회
  BYPASS  // 캐시를 무시하고 데이터베이스에 직접 접근
}
------------------------------------------------------------
//캐시 보관 모드
public enum CacheStoreMode {
  USE,        // 조회한 데이터를 캐시에 저장. 이미 캐시에 있으면 최신 상태로 갱신하지 않음. 
              // 트랜잭션 커밋시 등록 수정한 엔티티도 캐시에 저장
  BYPASS,     // 캐시에 저장하지 않는다
  REFRESH     // USE 전략에 추가로 데이터베이스에서 조회한 엔티티를 최신 상태로 다시 캐시
}
------------------------------------------------------------
//캐시 모
// 1. em.setProperty()
em.setProperty("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
em.setProperty("javax.persistence.cache.storeMode", CacheStoreMode.BYPASS);

// 2. em.find()
Map<String, Object> param = new HashMap<String, Object>();
param.put("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
param.put("javax.persistence.cache.storeMode", CacheStoreMode.BYPASS);
em.find(TestEntity.class, id, param);

// 3. Query.setHint()
em.createQuery("select a from TestEntity e where e.id = :id", TestEntity.class)
  .setParameter("id", id);
  .setHint("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
  .setHint("javax.persistence.cache.storeMode", CacheStoreMode.BYPASS);
  .getSingleResult();
```

**JPA 캐시 관리 API**

* JPA는 캐시를 관리하기 위한 javax.persistence.Cache 인터페이스를 제공

#### **하이버네이트와 EHCACHE 적용**

* 하이버네이트가 지원하는 캐시
  * 엔티티 캐시 / 컬렉션 캐시 / 쿼리 캐시 
* 환경설정 
  * pom.xml에 hibernate-ehcache 라이브러리 추가
  * src/main/resources/ehcache.xml 설정 파일 추가 
    * 캐시 보관 기간, 캐시 보관 크기 등의 캐시 정책 설정
  * persistence.xml에 캐시 사용정보 설정
* 부모는 엔티티 캐시 적용, 자식들은 컬렉션 캐시 적용
* 엔티티 캐시 영역은 \[패키지 명 + 클래스 명\]
* 컬렉션 캐시 영역은 \[패키지 명 + 클래스 명 + 필드 명\]의 기본값을 가진다.
* 쿼리 캐시는 활성화하면 두 캐시 영역이 추가된다.\(StandardQueryCache, UpdateTimestampsCache\) 캐시한 데이터 집합을 최신 데이터로 유지하려고 쿼리 캐시를 실행하는 시간과 테이블들이 가장 최근에 변경된 시간을 비교하여 변경이 있으면 데이터베이스에서 데이터를 읽어와 쿼리 결과를 다시 캐시 한다. 따라서 수정이 적은 테이블에 사용해야 효과적이다.

**쿼리 캐시와 컬렉션 캐시의 주의점**

* 엔티티 캐시는 엔티티 정보를 모두 캐시, 쿼리 캐시와 컬렉션 캐시는 결과 집합의 식별자 값만 캐시
* 쿼리 캐시와 컬렉션 캐시는 식별자 값을 하나씩 엔티티 캐시에서 조회해서 실제 엔티티를 찾는다
* 엔티티에 쿼리 캐시나 컬렉션 캐시만 사용하고 엔티티 캐시를 적용하지 않으면 결과 집합만큼 SQL을 실행하는 성능 문제가 발생할 수 있다
* 쿼리 캐시나 컬렉션 캐시 사용시 꼭 엔티티 캐시를 적용해야 함


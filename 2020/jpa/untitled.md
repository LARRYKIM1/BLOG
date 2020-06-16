# \#16 트랜잭션과 락, 2차캐시

## 1. 트랜잭션과 락 

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








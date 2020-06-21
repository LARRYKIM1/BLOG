---
description: '김영한 저 "자바 ORM 표준 JPA 프로그래밍, 2015"을 읽고 정리한 내용입니다.'
---

# \#15 예외 실습

## RollbackException 예외

```java
@Entity
public class Author {
    @Id @GeneratedValue
    private Long id;
    
    @Column(nullable = false, length = 50)
    private String firstName; // null 허용 안됨.
    
    private String lastName;
    
    @Column(length = 2000)
    private String bio;
    
    private String email;
    
    // Constructors, getters, setters
}
----------------------
Author author = new Author().firstName(null);
tx.begin();
em.persist(author);  // RollbackException 발생  
assertThrows(RollbackException.class, () -> tx.commit());
```

## EntityNotFoundException 예외

getReference\(\)는 상태데이터\(state data\)를 지연로딩한다. 즉, 엔티티가 비영속 되기 전에 상태에 접근하지 않는다면 데이터는 존재하지 않을것이다. 만약 엔티티가 찾을 수 없을때는 EntityNotFoundException이 발생한다.

```java
try {
    Customer customer = em.getReference(Customer.class, id);
    // Process the object
    assertNotNull(customer);
} catch (
    EntityNotFoundException ex) {
    // Entity not found
}
```

## IllegalStateException 예외

무결성 제약조건에 대한 문제  
고엔티티매니저는, 명시적 플러시를 하지 않으면, 모든 변경과 요구들을 캐싱하고 일관된 방법으로 데이터베이스에 실행 시킨다. 고객에 대한 INSERT는 실행되겠지만, 주소엔티티에 대한 상태\(State\)는 일관성되 않는다. 즉, 주소 엔티티는 아직 영속화가 되지 않았고 따라서, id값이 존재하지 않는다.

```java
Customer customer = new Customer("Anthony", "Balla", "aballa@mail.com");
Address address = new Address("Ritherdon Rd", "London", "8QE", "UK");
customer.setAddress(address);

assertThrows(IllegalStateException.class, () -> {
    tx.begin();
    em.persist(customer); //이때 address는 같이 영속화가 안되나? 
    em.flush();  // 성공
    em.persist(address);
    tx.commit();
});
```

## NonUniqueResultException NoResultException 예외

**`getSingleResult()`** 메소드 실행시 1개 보다 많은 결과가 찾아질 때 발생. 후자는 없을때.

## IllegalArgumentException 예외

매개변수 제대로 안넘어 올 때.

```java
@Entity
public class Customer {
    @Id @GeneratedValue
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String phoneNumber;
    private LocalDate dateOfBirth;
    @Transient
    private Integer age;
    private LocalDateTime creationDate;
    
    @PrePersist
    @PreUpdate
    private void validate() {
        if (firstName == null || firstName.isEmpty())
            throw new IllegalArgumentException("Invalid first name");
        if (lastName == null || lastName.isEmpty())
            throw new IllegalArgumentException("Invalid last name");
    }
    
    @PostLoad
    @PostPersist
    @PostUpdate
    public void calculateAge() {
        if (dateOfBirth == null) {
        age = null;
        return;
    }
        age = Period.between(dateOfBirth, LocalDate.now()).getYears();
    }
    
    // Constructors, getters, setters
}
```

## OptimisticLockException 

![Antonio Goncalves, &#x300C;Understanding JPA 2.2&#x300D;, Amazon KDP, 2019 ](../../../.gitbook/assets/image%20%2856%29.png)

## LockTimeoutException 예외

비관적 락을 사용할때,  무한정 기다릴 수 없으므로 타임아웃 시간을 줄 수 있다.

```java
Map<String,Object> properties = new HashMap<String,Object>();

//타임아웃 10초까지 대기 설정
properties.put ("javax.persistence. lock. timeout", 10000);
Board board = em.find(Board.class, "boardId",
        LockModeType.PESSIMISTIC_WRITE, properties);
```

## 참고문헌 

Understanding JPA 2.2 [책](https://www.amazon.com/Understanding-JPA-2-2-Persistence-fascicle-ebook/dp/B07RWPXPS6/ref=sr_1_2?dchild=1&keywords=JPA&qid=1592293308&sr=8-2)


---
description: 김영한 저 "자바 ORM 표준 JPA 프로그래밍"을 읽고 정리한 내용입니다.
---

# \#05 연관관계 매핑 기초

객체 연관관계와 테이블 연관관계를 매핑 어려운 일이니 차근히 알아보자. 

### 방향

* 단방향
  * 회원 -&gt; 팀 OR 팀 -&gt; 회원 둘 중 하나만 참고 하는 것
* 양방향
  * 회원 -&gt; 팀 AND 팀 -&gt; 회원 모두 서로 참조하는 것

###  다중성\(Multiplicity\)

* 다대일\(N:1\)
* 일대다\(1:N\)
* 일대일\(1:1\)
* 다대다\(N:M

### @JoinColumn

| 속성 | 기능 | 기본값 |
| :---: | :--- | :--- |
| name | 매핑할 외래 키 이름 | 언더스코어 표기법\(TEAM\_ID\) |
| referencedColumnName | 외래 키가 참조하는 대상 테이블의 컬럼명 | 참조하는 테이블의 기본키 컬럼명 |
| foreignKey\(DDL\) | 외래 키 제약조건을 직접 지정할 수 있다.   이 속성은 테이블을 생성할 때만 사용한다. |  |
| unique, nullable,   insertable, updateable   columnDefinition, table | @Column의 속성과 같다 |  |

### @ManyToOne

| 속성 | 기능 | 기본값 |
| :--- | :--- | :--- |
| optional | false로 설정하면 연관된 엔티티가 항상 있어야 함 | true |
| fetch | 글로벌 패치 전략 | @ManyToOne=FetchType.EAGER   @OneToMany=FetchType.LAZY |
| cascade | 영속성 전이 기능 |  |
| targetEntity | 연관된 엔티티의 타입 정보 설정 |  |

## 요약

### **\[1\] 객체 연관관계와 테이블 연관관계의 차이**

* 객체 - 참조를 통한 연관관계는 언제나 **단방향**\(객체 연관관계\) 
* 테이블 - 조인 가능한 **양방향** 관계

### **\[2\] 양방향 연관관계 시 주인\(owner\) 설정** 

* **외래 키가 있는 곳**을 기준\(**데이터베이스**는 ****1:N,  N:1 관계에서는 **항상 다 쪽이 외래 키**를 가진다.\)
* 연관관계의 주인만이 외래 키의 값을 변경할 수 있기 때문에, 주인이 아닌 곳에 저장한 값은 반영 X
* 객체 관점에서 양쪽 방향에 모두 값을 입력하지 않으면 JPA를 사용하지 않는 순수한 객체 상태에서 심각한 문제가 발생할 수 있으므로 모두 입력해주는 것이 가장 안전하다.

### **\[3\] 앙방향 매핑시 무한 루프에 유의**

* Member.toString\(\)에서 getTeam\(\) 호출, Team.toString\(\)에서 getMember\(\) 호출 시 무한 루프에 빠질 수 있다.

### \[4\] 편의 메소드

* 양방향 연관관계에서 member.setTeam\(team\), team.getMembers\(\).add\(member\) 를 각각 호출하게 되면 둘 중 하나만 호출 하는 실수에 대처하기 어렵다. Member 클래스의 setTeam\(\) 메소드를 수정해서 한번에 두 가지를 호출하도록 하는 것을 연관관계 편의 메소드라 한다.

## 코드 분석 

**&lt; Member.class &gt;**

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    private String city;
    private String street;
    private String zipcode;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<Order>();

    //Getter, Setter Omitted
   
}
```

**&lt; Order.class &gt;**

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;      //주문 회원

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<OrderItem>();

    @Temporal(TemporalType.TIMESTAMP)
    private Date orderDate;     

    @Enumerated(EnumType.STRING)
    private OrderStatus status; //주문상태 주문완료, 취소

    //==연관관계 메서드==//
    public void setMember(Member member) {
        //기존 관계 제거
        if (this.member != null) {
            this.member.getOrders().remove(this);
        }
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    //Getter, Setter Omitted
    
    //    ...

    @Override
    public String toString() {
        return "Order{" +
                "id=" + id +
                ", orderDate=" + orderDate +
                ", status=" + status +
                '}';
    }
}
```






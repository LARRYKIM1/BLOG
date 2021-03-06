---
description: '김영한 저 "자바 ORM 표준 JPA 프로그래밍, 2015"을 읽고 정리한 내용입니다.'
---

# \#06 다양한 연관관계 매핑

#### 엔티티 매핑시 고려사항 3가지 

* 다중성 ****- ManyToOne, OneToMany, ManyToOne, ManyToMany 고려
* 단방향, 양방향 - 참조 방향 고려
* 연관관계 주인 - 양방향일 경우 고려할 사항 

#### 팁

* 보통 외래키를 가진 테이블과 매핑한 엔티티가 외래키를 관리한다.
* 연관관계 주인은 mappedBy 속성 사용 안함

## 6.1 다대일\(ManyToOne\)

### 6.1.1 다대일 단방향\(unidirectional\)

### 6.1.2 다대일 양방향\(bidirectional\)

```java
@Entity
public class Member {
	@Id
	@GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	private String username;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;

	public void setTeam(Team team) {
		this.team = team;
		// 무한루프에 빠지지 않도록 체크
		if (!team.getMembers().contains(this)) {
			team.getMembers().add(this);
		}
	}
}
```

```java
@Entity
public class Team {
	@Id
	@GeneratedValue
	@Column(name = "TEAM—ID")
	private Long id;

	private String name;

	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<Member>();

	public void addMember(Member member) {
		this.members.add(member);
		if (member.getTeam() != this) {
			// 무한루프에 빠지지 않도록 체크
			member.setTeam(this);
		}
	}
}
```

양방향 연관관계는 항상 서로를 참조해야 함





## 6.2 일대다\(OneToMany\)

### 6.2.1 1:N 단방향\(unidirectional\)

```java
@Entity
public class Team {
	@Id
	@GeneratedValue
	@Column(name = "TEAM_ID")
	private Long id;

	private String name;

	@OneToMany
	@JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID(FK)
	private List<Member> members = new ArrayList<Member>();

	// Getter, Setter ...
}
```

```java
@Entity
public class Member {
	@Id
	@GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	private String username;
	// Getter, Setter ...
}
```

### 6.2.2 1:N 양방향\(bidirectional\)

```java
@Entity
public class Team {
	@Id
	@GeneratedValue
	@Column(name = "TEAM_ID")
	private Long id;

	private String name;

	@OneToMany
	@JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID(FK)
	private List<Member> members = new ArrayList<Member>();

	// Getter
}
```

```java
@Entity
public class Member {
	@Id
	@GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	private String username;

	@ManyToOne
	@JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
	private Team team;
	// Getter, Setter ...
}
```

## 6.3 일대일\(OneToOne\)

### 6.3.1 주 테이블에 외래 키

#### 단방향

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    ...
}
```

#### 양방향

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Member member;
    ...
}
```

### 6.3.2 대상 테이블에 외래 키

#### 단방향



#### 양방향 

```java
@Entity
public class Member{
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    
    private String username;
    
    @OneToOne(mappedBy = "member")
    private Locker locker;
    ...
}

@Entity
public class Locker{
    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    ...
}
```

## 6.4 다대다\(ManyToMany\)

### 단방향 

```java
@Entity
public class Member {
    @Id @Column(name = "MEMBER_ID")
    private String id;

   	private String username;

    @ManyToMany @JoinTable(name = "MEMBER_PRODUCT",
                           joinColumns = @JoinColumn(name = "MEMBER_ID"),
                           inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")) 
    private List<Product> products = new ArrayList<Product>();
    ...
}

@Entity
public class Product{
    @Id @Column(name = "PRODUCT_ID")
    private String id;
    private String name;
    ...
}
```

### 양뱡항 

```java
@Entity
public class Product{
    @Id @Column(name = "PRODUCT_ID")
    private String id;
    private String name;
    
    @ManyToMany(mappedBy = "products")
    private List<Member> members;
    ...
}
```

### 다대다 : 매핑의 한계와 극복, 연결 엔티티 사용

```java
@Entity
public class Member{
    @Id @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts;
    ...
}

@Entity
public class Product{
    @Id @Column(name = "PRODUCT_ID")
    private String id;
    private String name;
    ...
}

@Entity
@IdClass(MemberProductId.class)
public class MemberProduct{
	@Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    
    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
    
    private int orderAmount;
    ...
}

public class MemberProductId implements Serialiable {
    private String member; //MemberProduct.member와 연결
    private String product; //MemberProduct.product와 연결
    
    @Override
    public boolean equals(Object o){...}
    
    @Override
    public int hashCode(){..}
}
```

### 다대다 : 새로운 기본 키 사용












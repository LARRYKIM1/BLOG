---
description: '조영호 저 "오브젝트: 코드로 이해하는 객체지향 설계, 2019"를 읽고 정리한 내용입니다.'
---

# \# 13장 서브클래싱과 서브타이핑 - 발표

## 서브클래싱과 서브타이핑

이 두 용어는 다르다.

상속의 주된 용도

* 타입계층 구현
* 코드 재사용 - 강하게 결합 위험 높음.

타입 계층의 관점에서 부모클래스는 자식클래의 generalization 일반화이고 그 역은 specialization 특수화이다.

상속의 일차적인 목표는 코드 재사용이 아니라 **타입계층 구현**이어야 한다.

결론부터 말하면 동일한 메시지에 대해 서로 다르게 행동할 수 잇는 다형적인 객체를 구현하기 위해서는 객체의 **행동을 기반으로 타입 계층을 구성**해야 한다. 이따 oop에서의 타입에서 설명. 행동이 퍼블릭인터페이스이고 이것이 같으면 같은 타입으로 분류.

> oop obp 차이는 상속 다형성 지원여부이며 상태와 행동을 캡슐한 객체를 조합해서 프로그램을 구성한다는 면은 동일하다.

그럼 타입 계층이란 무엇인가?

## 1 타입

개념 관점의 타입 vs 프로그래밍 언어 관점의 타입

#### 프로그래밍 언어 관점의 타입

1 개념 관점의 타입: 우리가 인식하는 객체들에 적용하는 개념이나 아이디어를 가리킨다. 세상의 사물. 타입은 사물을 분류하기 위한 틀로 사용된다.

* 타입의 세가지 요소 - **symbol** 심볼 / **intension** 내연 / **extension** 외연

문제 - 다음은 어느 요소에 대한 설명인가?

* 타입에 이름을 붙인 것. 예, 프로그래밍 언어.
* 타입에 속하는 객체들이 가지는 공통적인 속성이나 행동. 예, 컴퓨터에게 특정한 작업을 지시하기 위한 어휘와 문법적 규칙의 집합.
* 타입에 속한 객체들의 집합. 예, 자바, 루비 등의 집합.

2 프로그래밍 언어 관점의 타입: 비트 묶음에 의미를 부여하기 위해 정의된 제약과 규칙이다. 예, 비트에 담긴 데이터를 문자열로 다룰지, 정수로 다룰지.

프로그래밍 언어 관점의 타입의 두가지 목적은

* 타입에 유효한 오퍼레이션의 집합을 정의한다. 예, 객체 타입에 따라 적용 가능한 연산자\(+\)의 종류를 제한 \(다른 클래스의 인스턴스에 대해서는 + 사용 불가\)
* 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공한다. 예, 1+1 더하고 s+s는 하나로 합친다.

#### 요약

개념 관점에서 타입이란 공통의 특징을 공유하는 대상의 분류이고, 프로그래밍 언어관점의 타입은 동일한 오퍼레이션을 적용할 수 있는 인스턴스들의 집합이다.

#### 객체지향 패러다임 관점의 타입

객체의 타입이란 객체가 수신할 수 있는 메시지의 종류를 정의하는 것. 이 메시지 집합은 퍼블릭 인터페이스이다. 즉, oop에서 타입을 정의한다는 것은 객체의 **퍼블릭 인터페이스를 정의하는 것**이다. 동일한 PI를 제공하는 객체들은 동일한 타입으로 분류된다.

## 2 타입 계층

#### 타입 사이의 포함관계

타입의 세가지 요소의 상세 설명..

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPJBslXx3rya5aK9l%2F-MCVP_tGizNyAW1YK3Hk%2Fimage.png?alt=media&token=3aca395c-047c-4304-8192-9d40de453605)

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPJBslXx3rya5aK9l%2F-MCVPZNUD3W1Z8jU8lEG%2Fimage.png?alt=media&token=5024fb1a-96f4-4103-b826-1f1de9359392)

문제 - 포함하는 타입은 외연 관점에서 더 크고 내연 관점에서는 더 일반적이다. 이것 의미를 서술하시오. 답, 프로그래밍 언어 타입 7개 인데 클래스 기반 언어 타입은 3개를 가짐.

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPJBslXx3rya5aK9l%2F-MCVPVhwtq_JYFONa3Oc%2Fimage.png?alt=media&token=a4343c4b-833d-4322-9916-d58f854ce5ee)

슈퍼 타입, 서브 타입이란 무엇일까?

답, 프로그래밍 언어 - 객체지향언어,절차적언어는 슈퍼타입

슈퍼타입과 서브타입을 쉽게 말해, 슈퍼타입은 더 추상적이고 더 일반적인 반면 반면 서브타입은 덜 추상적\(구체적\)이고 더 특수한\(구체적\) 퍼블릭 인터페이스를 가졌다.

> 일반화 / 특수화란 무엇일까?
>
> 일반화는 다른 타입을 완전히 포함하거나 내포하는 타입을 식별하는 행위 또는 그 행위의 결과를 가리킨다. 특수화는 다른 타입 안에 전체적으로 포함되거나 완전히 내포되는 타입을 식별하는 행위 또는 그 행위의 결과를 가리킨다

#### 객체지향 프로그래밍과 타입 계층

## 3 서브클래싱과 서브타이핑

### 언제 상속을 사용해야 하는가?

* 상속 관계가 is-a 관계
* 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 무방한가? \(우선시해야함\) 용어, 행동호환성.

 is-a 관계만 초점을 맞출경우 - 펭귄은 새다. 새는 날수 있다. 자식클래스가 부모클래스 대체 안되는 문제 \(행동호환성\).

```java
public class Bird {
    public void fly() { ... }
}
public class Penguin extends Bird {
    ...
}
```

#### 행동호환성

타이핑 기준은 사용하는 **클라이언트 기준**이 된다. 클라이언트는 모든 새가 날 수 있다 가정한다. 저렇게 타입계층 구성하면 안됨. 행동호환성 어김..

이문제 해결 3가지   
1 fly 메서드를 비워둔다. 하지만, 클라이언트는 아직도 모든 새가 날수 있다고 생각한다. 설계적 결함.   
2 펭귄 fly에서 예외를 던진다. 클라이언트입장 새-펭귄 호환 아직도 안됨.  
3 flyBird에 펭귄은 if문에서 제외. 지속적인 if문 문제.. OCP 위반

```java
public void flyBird(Bird bird){
    if (!(bird instanceof Penguin)){ bird.fly() }
}
```

#### 클라이언트의 기대에 따라 계층 분리하기

```java
public class Bird {
    ...
}
public class FlyingBird extends Bird {
    public void fly() { ... }
}
public class Penguin extends Bird {
    ...
}
//----------------------------------
public void flyBird(FlyingBird bird) {
    bird.fly();
}
```

모든 클래스들이 행동호환성을 만족한다.

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPgQetWh6XYWevMB7%2F-MCVPj-skUvHHhceRg91%2Fimage.png?alt=media&token=e5651689-a8ef-42a5-b806-fdb4517e46b9)

두개의 클라이언트가 있을 경우

모든 것은 각 클라이언트 **기대에 부응해서 인터페이스가 분리**되어야 한다.

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPgQetWh6XYWevMB7%2F-MCVPl7_o8TVn1x2_0Pj%2Fimage.png?alt=media&token=d87609f1-9b7e-428f-a55c-37c3b836521c)

만약 펭귄이 새의 코드를 재사용 하고 싶다면?

상속말고, 합성 사용.

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPgQetWh6XYWevMB7%2F-MCVPmru1FGyNJojXJbI%2Fimage.png?alt=media&token=6633a50f-e998-4472-9760-19e26f97a5c5)

클라이언트에 따라 인터페이스를 분리하면 변경에 대한 영향을 **세밀하게 제어**할 수 있다.

> 인터페이스 분리 원칙 ISP
>
> 비대한 인터페이스를 가지는 클래스는 응집성이 없다.
>
> [링크](https://velog.io/@amobmocmo/Software-Design-ISP-Interface-Segregation-Principle)

#### 서브클래싱과 서브타이핑

서브클래싱과 서브타이핑을 나누는 기준은 상속을 사용하는 목적이다.

서브클래싱\(=구현상속, 클래스상속\): **코드를 재사용**할 목적으로 상속을 하는 경우   
서브타이핑\(=인터페이스 상속\): 타입 **계층을 구성**하기 위해 상속을 하는 경우. 예, DiscountPolicy.

> 지우기 - 뒷부분
>
> Stack과 Vector가 서브타이핑 관계가 아니라 서브클래싱 관계인 이유도 마찬가지다. Stack과 Vector가 리스코프 치환 원칙을 위반하는 가장 큰 이유는 상속으로 인해 Stack에 포함돼서는 안 되는 Vector의 퍼블릭 인터페이스가 Stack의 퍼블릭 인터페이스에 포함됐기 때문이다. Stack의 행동은 Vector의 행동과 호환되지 않는다.
>
> ```java
> Stack<String> stack = new Stack<>();
> stack.push("1st");
> stack.push("2nd");
> stack.push("3rd");
> ​
> stack.add(0,"4th"); // Vector의 인터페이스 사용...
> ```

> 서브타이핑이 인터페이스 상속으로 용어 호환 이유는?
>
> 부모타입을 자식타입으로 대체가능해야한다. 이것은 자식의 퍼블릭 인터페이스가 부모의 것과 동일하거나 더 많은 오퍼레이션을 포함해야 한다는 것이다. 슈퍼타입의 퍼블릭 인터페이스를 상속 받는 것처럼 보여 서브타이핑을 인터페이스 상속이라고 부르기도 한다.

> **클래스 상속**은 객체의 구현을 정의할 때 이미 정의된 객체의 구현을 바탕으로 한다. 즉, 코드 공유의 방법이다. 이에 비해 **인터페이스 상속**\(서브 타이핑\)은 객체가 다른 곳에서 사용될 수 있음을 의미한다. 인터페이스 상속 관계를 갖는 경우 프로그램에는 슈퍼티입으로 정의하지만 런타임에 서브타입의 객체로 대체할 수 있다.
>
> 대부분의 프로그래밍 언어들은 인터페이스와 구현 상속을 구분하고 있지 않지만, 프로그래머들은 실제로 구분해서 사용하고 있다.
>
> 추상 클래스를 상속한다는 것은 단순한 코드의 재사용을 위한 상속이 아니라 추상 클래스가 정의하고 있는 인터페이스를 상속하겠다는 의미이다.

**행동호환성과 대체가능성**은 올바른 상속 관계를 구축하기 위해 따라야할 지침이다. 오랜 시간 동안 이 지침은 **리스코프 치환 원칙이라는 이름으로** 사용되어 소개되어 왔다.

is-a관계와 행동호환성을 리스코프 치환 원칙을 통해 다시 한번 정리하자.

## 4 리스코프 치환 원칙

리스코프 따르면, 서브타이핑 관계를 만족시키기 위한 조건은 S형의 각 객체 o1에 대해 T형의 객체 o2가 하나 있고, T에 의해 정의된 모든 프로그램 P에서 T가 S로 치환될 때, P의 동작이 변하지 않으면 S는 T의 서브타입이다. **= 서브타입은 기반 타입에 대해 대체 가능하다.**

예, Vector-Stack, Vector에 대해 기대하는 행동을 Stack에게는 기대 할 수 없다. 새-팽귄도 같음.

전형적인 위반 케이스

```java
public class Rectangle {
    private int x, y, width, height;
    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = heigh;
    }
    public int getWidth() {
        return width;
    }
    public void setWidth(int width) {
        this.width = width;
    }
    public int getHeight() {
        return height;
    }
    public void setHeight(int height) {
        this.height = height;
    }
    public int getArea() {
        return width * height;
    }
}
​
public class Square extends Rectangle {
    public Square(int x, int y, int size) {
        super(x, y, size, size);
    }
    @Override
        public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }
    @Override
        public void setHeight(int height) {
        super.setwidth(height);
        super.setHeight(height);
    }
}
​
// 클라이언트는 직사각형의 너비와 높이가 다르다고 가정한다.
// 직사각형의 너비와 높이를 서로 다르게 설정하도록 프로그래밍할 것이다
public void resize(Rectangle rectangle, int width, int height) {
    rectangle.setWidth(width);
    rectangle.setHeight(height);
    assert rectangle.getWidth() == width && rectangle.getHeight() == height;
}
​
Square square = new Square(10, 10, 10);
resize(square, 50, 100); // 결과적으로 실패
```

업캐스팅이 되어야 하지만 resize 관점에서 직사각형 대신 정사각형을 **대체 사용할 수 없다.**

is-a관계 뿐만 아니라 **행동호환성**도 중요하다는 것을 다시 깨달았다.

#### 클라이언트와 대체 가능성

행동 호환성과 리스코프 치환 원칙에서 한 가지만 기억해야 한다면 이것을 기억하라. 대체 가능성을 **결정하는 것은 클라이언트**다.

#### is-a 관계 다시 살펴보기

결론적으로 상속이 **서브타이핑을 위해** 사용될 경우에만 is-a 관계다. **서브클래싱을 구현하기 위해** 상속을 사용했다면 is-a 관계라고 말할 수 없다.

#### 리스코프 치환 원칙은 유연한 설계의 기반이다

클라이언트의 코드를 변경하지 않고도 새로운 자식 클래스와 협력할 수 있다. 아래 **Overlapped 정책 추가.**

![1](https://gblobscdn.gitbook.com/assets%2F-M7KQQ2ZLP5HVNbeQNZi%2F-MCVPgQetWh6XYWevMB7%2F-MCVPtBlU-5G1mVY15Q0%2Fimage.png?alt=media&token=e85bc1cb-bb0c-4e83-946b-77aa4c64dbee)

또한, 위 설계는 의존성 역전 원칙과 개방-폐쇄 원칙, 리스코프 치환원칙이 한데 어우러져 설계를 확장하게 만든 예이다.

문제 - DIP, OCP, LSP가 어디 있는지 설명해보자.

* DIP: 상위 수준의 모듈인 Movie와 하위 수준의 모듈인 OverlappedDiscountPolicy는 모두 추상 클래스인 DiscountPolicy에 의존한다.
* LSP: Discountpolicy 대신 OverlappedDiscountPolicy와 협력하더라도 아무런 문제가 없다. 다시 말해서 OverlappedDiscountPolicy는 클라이언트에 대한 영향 없이도 DiscountPolicy를 대체할 수 있다.
* OCP: 중복 할인 정책이라는 새로운 기능을 추가하기 위해 DiscountPolicy의 자식 클래스인 OverlappedDiscountPolicy를 추가하더라도 Movie에는 영향을 끼치지 않는다. 다시 말해서 기능 확장을 하면서 기존 코드를 수정할 필요는 없다.

LSP는 OCP의 전제 조건이다.

### 타입 계층과 리스코프 치환 원칙

잊지 말아야 하는 사실은 클래스 **상속은 타입 계층을 구현**할 수 있는 다양한 **방법 중 하나**일 뿐 이라는 것이다.

마지막 질문만이 남았다. 클라이언트 관점에서 자식 클래스가 부모 클래스를 대체할 수 있다는 것은 무엇을 의미하는가? 클라이언트 관점에서 자식 클래스가 부모 클래스의 행동을 보존한다는 것은 무엇을 의미하는가?

## 5 계약에 의한 설계와 서브타이핑

클라이언트와 서버 사이의 협력을 의무\(obligation\)와 이익\(benefit\)으로 구성된 계약의 관점에서 표현하는 것을 계약에 의한 설계\(Design By Contract, DBC\)라고 부른다.

세가지 요소

* precondition 사전 조건 / postcondition 사후 조건 / class invariant 클래스 불변식

precondition 클라이언트가 정상적으로 메서드를 실행하기 위해 만족시켜야 하는것,   
postcondition 메서드가 실행된 후에 서버가 클라이언트에게 보장해야 하는것,   
class invariant 메서드 실행 전과 실행 후에 인스턴스가 만족시켜야 하는 클래스 불변식

> 부록 설명 요약해 넣기

 **계약에 의한 설계를 사용하면** 리스코프 치환 원칙이 강제하는 조건을 계약의 개념을 이용해 좀 더 명확하게 설명할 수 있다.

> 서브타입이 리스코프 치환 원칙을 만족시키시 위해서는 클라이언트와 슈퍼타입 간에 체결된 계약을 준수해야 한다.

영화와 정책 사이 암묵적인 사전조건과 사후조건이 존재한다.

```java
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
​
public abstract class DiscountPolicy {
    public Money calculateDiscountAmount(Screening screening) {
        for(Discountcondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return screening.getMovieFee();
    }
    abstract protected Money getDiscountAmount(Screening screening);
}
```

사전조건: calculateDiscountAmount의 screening에 null이 전달된다면 screening, getMovieFee\(\)가 실행될 때 NullPointerException 예외가 던져질 것이다. 따라서 단정문\(assertion\)을 사용해 사전조건을 다음과 같이 표현할 수 있다.

```java
assert screening != null && screening.getStartTime().isAfter(LocalDateTime.now());
```

사후조건: calculateDiscountAmount 메서드의 반환값은 항상 null이 아니어야 한다. 추가로 반환되는 값은 청구되는 요금이기 때문에 최소한 0원보다는 커야 한다.

```java
assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);
```

두 조건 추가 코드 

```java
​public abstract class DiscountPolicy {
    public Money calculateDiscountAmount(Screening screening) {
        checkPrecondition(screening);
​
        Money amount = Money.ZERO;
        for(Discountcondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                amount = getDiscountAmount(screening);
                checkPostcondition(amount);
                return amount;
            }
        }
        amount = screening.getMovieFee();
        checkPostcondition(amount);
        return amount;
    }
​
    protected void checkprecondition(Screening screening) {
        assert screening 1= null && screening.getStartTime().isAfter(LocalDateTime.now());
    }
    protected void checkPostcondition(Money amount) {
        assert amount != null && amount.isGreaterThanOrEqual(Money.ZERO);
    }
​
    abstract protected Money getDiscountAmount(Screening screening);
}
​
```

Movie가 지켜야할 사전 조건 추가한다. calculateDiscountAmount 메서드가 정의한 사전조건을 만족시키는 것은 Movie의 책임이다. 위반하는 screening을 전달해서는 안 된다.

```java
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        if (screening == null || screening.getStartTime().isBefore(LocalDateTime.now())) {
            throw new InvalidScreeningException();
        }
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

> Movie의 입장에서 AmountDiscountPolicy, PercentDiscountPolicy 등 클래스들은 DiscountPolicy를 대체할 수 있기 때문에 서브타이핑 관계라고 할 수있다.

### 서브타입과 계약

새로운 BrokenDiscountPolicy 추가 - 자정 넘어 시작하는 것은 예매 불가 사전조건 가짐.

```java
public class BrokenDiscountPolicy extends DiscountPolicy {

    public BrokenDiscountPolicy(DiscountCondition... conditions) {
        super(conditions);
    }

    @Override
    public Money calculateDiscountAmount(Screening screening) {
        checkPrecondition(screening);                 // 기존의 사전조건
        checkStrongerPrecondition(screening);         // 더 강력한 사전조건

        Money amount = screening.getMovieFee();
        checkPostcondition(amount);                   // 기존의 사후조건
        checkStrongerPostcondition(amount);           // 더 강력한 사후조건
        return amount;
    }

    private void checkStrongerPrecondition(Screening screening) {
        assert screening.getEndTime().toLocalTime()
                .isBefore(LocalTime.MIDNIGHT);
    }

    private void checkStrongerPostcondition(Money amount) {
        assert amount.isGreaterThanOrEqual(Money.wons(1000));
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;
    }
}
```

Movie가 오직 DiscountPolicy의 사전조건만 알고 있다는 점이 문제이다**. Movie는 DiscountPolicy가 정의 하고 있는 사전조건을 만족시키기 위해** null이 아니면서 시작시간이 현재 시간 이후인 **Screening을 전달**할 것이다. 따라서 자정이 지난 후에 종료되는 Screening을 전달하더라도 **문제가 없다고 가정**할 것 이다. **협력 실패**. **BrokenDiscountPolicy**는 클라이언트의 관점에서 **DiscountPolicy를 대체할 수 없기 때문**에 서브타입이 아니다.

즉, 서브타입에 **더 강력한 사전조건을 정의할 수 없다.**

그럼 사전조건을 제거해서 약화시킨다면? 이 경우는 상관없다.

사후조건을 강화한다면 어떨까? 또한 상관없다.

```java
// 클라이언트는 0원 이상만 받환 받으면 된다.
private void checkStrongerPostcondition(Money amount) {
    assert amount.isGreaterThanOrEqual(Money.wons(1000));
}
```

사후조건을 약하게 정의한다면? 문제가 된다... 마이너스 반환...

```java
private void checkWeakerPostcondition(Money amount) {
    assert amount != null;
}
```

계약에 의한 설계는 클라이언트 관점에서의 대체 가능성을 계약으로 설명할 수 있다는 사실을 잘 보여 준다. 따라서 서브타이핑을 위해 상속을 사용하고 있다면 부모 클래스가 클라이언트와 맺고 있는 계약에 관해 깊이 있게 고민해야 한다.

> 상속말고도 타입계층 구현 보고 싶으면, 부록B@@ 보기

결론? 요구사항을 얼마나 잘 분석해서 상속구조를 잘 만들어 낼것인가에 대한 설명과 주의사항


---
description: '조영호 저 "오브젝트: 코드로 이해하는 객체지향 설계, 2019"를 읽고 정리한 내용입니다.'
---

# \# 11장 합성과 유연한 설계

## 1 상속을 합성으로 변경하기

* 전 장에서 본 상속 문제
  * 불필요한 인터페이스 상속 문제  - Properties와 Stack
  * 메서드 오버라이딩의 오작용 문제 - InstrumentedHashSet
  * 부모 클래스와 자식클래스의 동시 수정 문제 

#### 1 불필요한 인터페이스 상속 문제

```java
// 합성 사용 Properties is-not-a Hashtable
public class Properties {
    private Hashtable<String, String> properties = new Hashtable <>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}

// 합성 사용 Stack is-not-a Vector
public class  Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);
        return item;
    }

    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```

#### 2 메서드 오버라이딩의 오작용 문제

```java
public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;

    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    // 나머지 Set 오버라이딩 생략
}
```

#### 3 부모 클래스와 자식클래스의 동시 수정 문제

```java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();

    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());
    }

    public List<Song> getTracks() {
        return tracks;
    }

    public Map<String, String> getSingers() {
        return singers;
    }
}

public class Song {
    private String singer;
    private String title;

    public Song(String singer, String title) {
        this.singer = singer;
        this.title = title;
    }

    public String getSinger() {
        return singer;
    }

    public String getTitle() {
        return title;
    }
}

public class PersonalPlaylist {
    private Playlist playlist = new Playlist();

    public void append(Song song) {
        playlist.append(song);
    }

    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
```

> #### 몽키패치
>
> 현재 실행중인 환경에만 영향을 미치도록 지역적으로 코드를 수정하거나 확장하는 것이다. 자바에서는 몽키패치를 지원하지 않아 바이트코드를 직접 변환하거나 AOP를 이용해 구현한다.

## 2 상속으로 인한 조합의 폭발적인 증가

상속으로 인해 결합도가 높아지면 코드를 수정하는데 필요한 작업의 양이 과도하게 늘어나는 경향이 있다. 작은 기능들을 조합해서 더 큰 기능으로 수행하는 객체를 만들어야 하는 경우가 해당된다. 이 경우 두가지 문제가 발생한다.

* 하나의 기능을 추가하거나 수정하기 위해 불필요하게 많은 수의 클래스를 추가하거나 수정해야 한다.
* 단일 상속만 지원하는 언어에서는 상속으로 인해 오히려 중복 코드의 양이 늘어날 수 있다.

#### 기본정책과 부가정책 조합하기

![](../../../.gitbook/assets/image%20%28135%29.png)

기본과 부가의 조합 경우의 수가 많을 것이다.

#### 상속을 이용해서 기본정책 구현하기

코드는 아래있다.

![](../../../.gitbook/assets/image%20%28137%29.png)

#### 기본정책에 세금정책 조합하기

> 훅메서드란?
>
> 추상메서드와 동일하게 자식 클래스에서 오버라이딩할 의도로 메서드를 추가했지만 편의를 위해 기본구현을 제공하는 메서드

#### 중복 코드의 덫에 걸리다

확장을 하다보니 위의 UML이 아래와 같이 변햇다..

![](../../../.gitbook/assets/image%20%28130%29.png)

무리한 상속 \(코드는 두경우만 가져와봤다\)  
Phone - RegularPhone - TaxableRegularPhone   
Phone - NightlyDiscountPhone - TaxableNightlyDiscountPhone 

```java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;

        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }

        return afterCalculated(result);
    }

    protected Money afterCalculated(Money fee) {
        return fee;
    }

    protected abstract Money calculateCallFee(Call call);
}
---------------------------------------------------------
public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;

    public RegularPhone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;

    public TaxableRegularPhone(Money amount, Duration seconds, double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
---------------------------------------------------------
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}


public class TaxableNightlyDiscountPhone extends NightlyDiscountPhone {
    private double taxRate;

    public TaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
        super(nightlyAmount, regularAmount, seconds);
        this.taxRate = taxRate;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

## 3 합성 관계로 변경하기

상속관계는 컴파일 타임에 결정되고 고정되기 때문에 코드를 실행하는 도중에는 변경할 수 없다.

#### 기본 정책 합성하기

코드는 맨아래 있다.

![](../../../.gitbook/assets/image%20%28132%29.png)

#### 부가 정책 적용하기

![](../../../.gitbook/assets/image%20%28136%29.png)

#### 기본 정책과 부가 정책 합성하기

#### 새로운 정책 추가하기

![](../../../.gitbook/assets/image%20%28134%29.png)

#### 객체 합성이 클래스 상속보다 더 좋은 방법이다.

상속 = 부모-자식 강하게 결합.   
합성 = 코드 재사용 + 건전한 결합

상속 2가지 - 구현 상속 / 인터페이스 상속

이번장의 상속에 대한 모든 단점들은 **구현 상속에 국한**된다. 인터페이스 상속은 좋다는 얘기인거 같다.

코드를 재사용할 수 있는 한가지 기법을 더 살펴보자. **믹스인**. 상속과 합성의 특성을 모두 보유한 톡특한 방법.

## 4 믹스인 = abstract subclass

객체를 생성할 때 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법. 믹스인은 코드 재사용에 특화된 방법이면서 상속과 같은 결합도 문제를 초래하지 않는다.

스칼라 예제.

스칼라의 trait는 CLOS에서 \(믹스인 첫도입 언어\) 제공했던 믹스인의 기본철학을 가장 유사한 형태로 재현하고 있다.

{% embed url="https://github.com/eternity-oop/object/tree/master/chapter11/src/main/scala/org/eternity/billing" %}

#### 기본 정책 구현하기

#### 트레이드로 부가 정책 구현하기

#### 부가 정책 트레이드 믹스인 하기

![](../../../.gitbook/assets/image%20%28129%29.png)

![](../../../.gitbook/assets/image%20%28127%29.png)

#### 쌓을 수 있는 변경

BasicRatePolicy의 calculateFee 메서드의 **기능을 확장**하기 위해 믹스인을 사용 해봤다. **믹스인은** 상속 계층 안에서 확장한 클래스보다 **더 하위에 위치**한다. 다시말해 대상 클래스의 **자식 클래스처럼 사용될 용도**로 만들어진 것이다.

> 서브클래스에서는 슈퍼클래스를 명시해야 하지만 반대는 그렇지 않기에 비대칭적이다. 그래서 믹스인은 슈퍼클래스로부터 상속될 클래스를 명시하는 매커니즘을 표현한다.

믹스인 특징 - stackable modification

스칼라에서 트레이트는 코드 재사용 근간을 이루는 단위다. 트레이트로 메서드와 필드 정의를 캡슐화하면 트레이트를 조합한 클래스에서 그 메서드나 필드를 재사용할 수 있다. 클래스와 트레이트의 차이는, 클래스는 super 호출을 정적으로 바인딩하지만, 트레이트는 동적으로 바인딩한다. 그래서 특징인 stackable modification이 가능해진다.

[참고](https://stackoverflow.com/questions/17987704/an-example-of-a-mixin-in-java)

자바에서 Comparable가 상호 비교가능한 객체들의 정렬된 인스턴스 선언을 허락하는 믹스인 인터페이스이다. 선택적인 기능을 허락하기에 믹스인이라 불린다. 참고로, 추상클래스는 믹스인 정의가 안된다. **뭔말?**


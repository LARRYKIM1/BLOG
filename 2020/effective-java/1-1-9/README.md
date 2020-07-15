---
description: 'Joshua Bloch 저  이복연 역  「Effective Java 3rd Edition, 2018」를 읽고 정리하였습니다.'
---

# \[1주차\] 1-6

## 아이템 01: 생성자 대신 정적 패토리 메서드를 고려하라. - 발표

클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. 그 외에 기법이 하나 더 있다. 정적 팩토리 메서드\(static factory method\)이다.

#### 정적 팩터리 메서드에 흔히 사용하는 명명 방식들

`from` `of` `valueOf` `getInstance` `newInstance` `getType` `newType` `type`

* Date d = Date.from\(instant\);

```java
Instant instant = Instant.now();
Date date = Date.from(instant);
System.out.println(date);
```

* Set faceCards = EnumSet.of\(JACK, QUEEN, KING\);

```java
enum days {
   SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
}
------------------------------------------------------------
Set<days> set = EnumSet.of(days.TUESDAY, days.WEDNESDAY);

Iterator<days> iter = set.iterator();
while (iter.hasNext())
   System.out.println(iter.next());
```

* BigInteger prime = BigInteger.valueOf\(Integer.MAX\_VALUE\);
* StackWalker luke = StackWalker.getInstance\(options\);

```java
Calendar mycal1 = Calendar.getInstance();
Calendar mycal2 = Calendar.getInstance();
mycal2.set(1996, 9 , 23);
System.out.println("mycal1 :" + mycal1.getTime());
System.out.println("mycal2 :" + mycal2.getTime());
if(mycal1.equals(mycal2))
{
    System.out.println("두개 인스턴스는 동일합니다.");
}
else{
    System.out.println("두개 인스턴스는 같지 않습니다.");
}
```

* Object newArray = Array.newInstance\(classObject, arrayLen\);

```java
String[][] strarr = (String[][]) Array.newInstance(String.class, 2,2);

Array.set(strarr[0], 0, "javaTpoint");
Array.set(strarr[1], 1, ".Net");

System.out.println("Array[0][0] = " + Array.get(strarr[0], 0));
System.out.println("Array[1][1] = " + Array.get(strarr[1], 1));
```

* FileStore fs = Files.getFileStore\(path\);

```java
Path path= Paths.get("D:\\effectivejava.txt");

try {

    FileStore fs
            = Files.getFileStore(path);

    System.out.println("드라이버명: " + fs.name());
    System.out.println("파일시스템: " + fs.type());
    System.out.println("전체 공간: \t\t" + fs.getTotalSpace() + " 바이트");
    System.out.println("사용 중인 공간: \t" + (fs.getTotalSpace() - fs.getUnallocatedSpace()) + " 바이트");
    System.out.println("사용 가능한 공간: \t" + fs.getUsableSpace() + " 바이트");

}
catch (IOException e) {
    e.printStackTrace();
} 
```

* BufferedReader br = Files.newBufferedReader\(path\);
* List litany = Collections.list\(legacyLitany\);

#### 장점과 단점

#### 장점 1. 이름을 가질 수 있다.

```java
//"값이 소수인 Biginteger를 반환한다"는 의미를 더 잘 설명할 것 같은지 생각해보자.
BigInteger bi1 = new BigInteger(10, new Random());
BigInteger bi2 = BigInteger.probablePrime(10, new Random());

System.out.println("bi1 = " + bi1);
System.out.println("bi2 = " + bi2);
//요약, 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면，
//생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주자.
```

#### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

```java
//대표적인예 - 객체를 아예 생성하지 않는다.
Boolean.valueOf(Boolean.FALSE)

//아마 아래는 객체를 생성하는데 위에는 생성하지 않는것 같다...
//new Boolean(Boolean.TRUE)
```

반복되는 요청에 같은 객체를 반환하는 식으로 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다. \(=instance-controlled class\) 이로써 싱글턴\(아이템 3 참고\)으로 만들 수도, 인스턴스화 불가\(noninstantiable, 아이템 4 참고\)로 만들 수도 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다\(아이템 17 참고\). 인스턴스 통제는 플라이웨이트 패턴의 근간이 되며, 열거 타입\(아이템 34 참조\)은 인스턴스가 하나만 만들어짐을 보장한다.

#### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

* 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 준다.
* API를 만들 때, 이 유연성을 응용하면 **구현 클래스를 공개하지 않고**도 그 객체를 반환할 수 있어 **API를 작게 유지**할 수 있다. 인터페이스 기반 프레임워크 \(아이템 20\)를 만드는 핵심 기술이다.
* 자바 8 전에는 인터페이스에 정적 메서드를 선언할 수 없었다. 그렇기 때문에 이름이 Type인 인터페이스를 반환하는 정적 메서드가 필요하면, Types라는 \(인스턴스화 불가인\) 동반 클래스\(companion class\)를 만들어 그 안에 정의하는 것이 관례였다.

```java
//자바 8 이전
interface 인터페이스명{
   static 메소드명(); //정적 메소드 불가
}

class 동반클래스명{
   //인터페이스 필요한 정적메소드를 정의
}
```

* 자바 8부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸다.

#### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

* 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
* 가령 EnumSet 클래스\(아이템 36\)는 public 생성자 없이 오직 정적 팩터리만 제공하는데, OperJDK에서는 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다. \(대다수에 해당하는\) 원소가 64개 이하면 원소들을 long변수 하나로 관리하는 RegularEnumSet의 인스턴스를, 65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다.

```java
//인터넷 팩토리 패턴 참고하였슴..
public static void main(String[] args) {	
      ShapeFactory shapeFactory = new ShapeFactory();

      Shape shape1 = shapeFactory.getShape("CIRCLE");
      shape1.draw();

      Shape shape2 = shapeFactory.getShape("RECTANGLE");
      shape2.draw();

      Shape shape3 = shapeFactory.getShape("SQUARE");
      shape3.draw();
}
------------------------------------------
public class ShapeFactory {
	
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }		
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
         
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
         
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      
      return null;
   }
}
------------------------------------------
public interface Shape {
	void draw();
}
------------------------------------------
//동그라미
public class Circle implements Shape{
	@Override
	public void draw() {
		System.out.println("동그라미를 그린다...");
	}
}
//직사각형
public class Rectangle implements Shape{
	@Override
	public void draw() {
		System.out.println("직사각형를 그린다...");
	}
}
```

#### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

* 서비스 제공자 프레임워크\([service provider framework](https://www.baeldung.com/java-spi)\)를 만드는 근간이 된다.
* 예를 들어, JDBC가 있고, 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여，클라이언트를 구현체로부터 분리해준다.

#### 단점

* 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
* 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  * 생성자처럼 API설명에 명확히 드러나지 않으니, 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.

#### 요약

* 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니, 상대적인 장단점을 이해하고 사용하는 것이 좋다.
* 그런데... 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.

## 아이템 02: 생성자에 매개변수가 많다면 빌더를 고려하라.

### 1. 점층적 생성자 패턴

점층적 생성자 패턴\(**telescoping constructor pattern**\) 영양분 정보를 가진 객체를 만들어보자.

```java
public class NutritionFacts {
   private final int servingSize;
   private final int servings;
   private final int calories;
   private final int fat;
   private final int sodium;
   private final int carbohydrate;

   public NutritionFacts(int servingSize, int servings) {
      this(servingSize, servings, 0);
   }
   
   public NutritionFacts(int servingSize, int servings,
   int calories) {
      this(servingSize, servings, calories, 0);
   }
   
   public NutritionFacts(int servingSize, int servings,
   int calories, int fat) {
      this(servingSize, servings, calories, fat, 0);
   }
   
   public NutritionFacts(int servingSize, int servings,
   int calories, int fat, int sodium) {
      this(servingSize, servings, calories, fat, sodium, 0);
   }

  public NutritionFacts(int servingSize, int servings,
   int calories, int fat, int sodium, int carbohydrate) {
      this.servingSize = servingSize;
      this.servings = servings;
      this.calories = calories;
      this.fat = fat;
      this.sodium = sodium;
      this.carbohydrate = carbohydrate;
   }
}
```

매개변수 6개 조합을 일일이 만들어 줘야되 코드가 너무 길어진다. 그리고 지금은 매개변수가 6개 밖에 없지만 많아질 수록 더욱 복잡해질 것이다.

### 2. 자바빈즈 패턴

 위와 같은 상황을 대응하기 위해서 자바 빈즈패턴 사용을 고려할 수 있다.

```java
public class NutritionFacts {

   private int servingSize = -1; 
   private int servings = -1; 
   private int calories = 0;
   private int fat = 0;
   private int sodium = 0;
   private int carbohydrate = 0;
   
   public NutritionFacts() { }

   // Setters
   public void setServingSize(int val) { servingSize = val; }
   public void setServings(int val) { servings = val; }
   public void setCalories(int val) { calories = val; }
   public void setFat(int val) { fat = val; }
   public void setSodium(int val) { sodium = val; }
   public void setCarbohydrate(int val) { carbohydrate = val; }
}

// 생성
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

인스턴스를 만들기 더 쉬워졌고, 필수 매개변수와 선택 매개변수를 구분해서 받아들일 수 있다. 하지만 자바빈즈의 경우에도 심각한 단점이 하나 있다. 바로 객체의 불변성을 보장하지 못해 객체의 일관성이 훼손될 수도 있다. 

### 3. 빌더 패턴

점층적 생성자 패턴의 안전성 장점과, 자바 빈즈 패턴의 가독성 장점을 모두 가져온 빌더패턴을 알아보자.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder {
        // 필수 파라미터들
        private final int servingSize;
        private final int servings;
        // 선택할 수 있는 파라미터들 
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) { calories = val; return this; }
        public Builder fat(int val) { fat = val; return this; }
        public Builder sodium(int val) { sodium = val; return this; }
        public Builder carbohydrate(int val) { carbohydrate = val; return this; }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    
    }
    
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}


// 사용 
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                        .calories(100).sodium(35).carbohydrate(27).build();
                        
// Builder에는 필수 파라미터 2개를 넣었고,
// setter로 선택적 파라미터를 넣어줬다.
```

Setter를 보면 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다\(메서드 체이닝\). 

유효성 검사 코드는 생략했되었지만, **잘못된 매개변수를 최대한 일찍 발견**하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식\(invariant\)을 검사하자.

기간을 표현하는 Period 클래스에서 start 필드의 값은 반드시 end 필드의 값보다 앞서야 하므로, 두 값이 역전되면 불변식이 깨진 것이다.

롬복의 @Builder 어노테이션을 사용하면 다 작성해 줄 필요가 없다.

```java
@Getter
@Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
}
```

#### 빌더 패턴은 계충적으로 설계된 클래스와 함께 쓰기에 좋다.

```java
NyPizza newyorkPizza =
        new NyPizza.Builder(LARGE) //필수 Builder 매개변수
                    .addTopping(HAM)
                    .addTopping(ONION).build();
Calzone calzonePizza =
        new Calzone.Builder()
                    .addTopping(HAM)
                    .sauceInside().build();

System.out.println("주문하신 피자");
System.out.println(newyorkPizza.toString());
System.out.println(calzonePizza.toString());
------------------------------------------------------------
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    //재귀적 타입 한정(아이템 30)을 이용하는 제네릭 타입
    //Builder타입의 자식클래스를 받는 Builder
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        abstract Pizza build();

        //하위 클래스는 이 메서드를 오버라이딩(재정의)하여
        //this를 반환해야 한다.
        //추상 메서드인 self를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다.
        //self 타입 이 없는 자바를 위한 이 우회 방법 (= simulated self-type)
        protected abstract T self();
    }
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
------------------------------------------------------------
//뉴욕 피자
public class NyPizza extends Pizza{
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        public Builder(Size size) { //사이즈 필수 매개변수
            this.size = Objects.requireNonNull(size);
        }
        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            //공변 반환 타이핑(covariant return typing)
            //클라이언트가 형 변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.
            return this;
        }
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    // toString() 생략
}
------------------------------------------------------------
//칼초네 피자
public class Calzone extends Pizza {
    private final boolean sauceInside;
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
        @Override public Calzone build() {
            return new Calzone(this);
        }
        @Override
        protected Builder self() {
            //공변 반환 타이핑(covariant return typing)
            //클라이언트가 형 변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.
            return this;
        }
    }
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    // toString() 생략
}
```

### 정리 

빌더 패턴과 자바 빈즈 패턴의 가장 큰 차이점은 불변성에 있다. 

자바 빈즈 패턴은 객체를 생성한 후, 값을 setter 메서드를 통해 넣는다. 그렇기에 객체 사용 도중 실수로, 혹은 악의적인 목적으로 setter 메서드를 통해 유효하지 않은 값이나 null값, 혹은 정확하지 않은 값이 들어갈 수 있다.

반면, 빌더 패턴은 객체 생성 전, 값을 setter 메서드를 통해 넣는다. 그리고 다 넣었다면 마지막에 build 메서드를 호출하여 객체를 생성한다. 그렇기 때문에 객체 사용 중에 값이 변경될 우려가 없으며, 불변성과 안정성이 올라간다. 당연하지만, 빌더 패턴 사용시에는 public setter 메서드를 선언해서는 안된다.

단점이 있다면, 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심하자. 그러니 애초에 빌더로 시작하는 편이 났다.

## 아이템 03: private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

싱글턴의 전형적인 예로는 함수\(아이템 24\)와 같은 무상태 \(stateless\) 객체나 설계상 유일해야 하는 시스템 컴포넌트를 들수 있다.

**싱글턴**을 만드는 방식은 3**가지**가 있다.

두 방식 모두 **생성자**는 **private**으로 **감춰두고**, 유일한 인스턴스에 접근할 수 있는 수단으로 **public static 멤버**를 하나 **마련해둔다.**

```java
// 첫번째 - public 필드 방식
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}

// 두번째 - 정적 팩터리 방식
// 정적 팩터리 메서드를 public static 멤버로 제공한다. getInstance()
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
// Elvis.getInstance()는 항상 같은 객체의 참조를 반환

// 세번째 - 열거 타입 방식
// 원소가 하나인 열거 타입을 선언
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

Elvis\(\) 생성자는 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다. 따라서 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

예외가 한가지 있는데, 권한이 있는 클라이언트는 **리플렉션 API**\(아이템 65\)인 **`AccessibleObject.setAccessible`**을 사용해 private 생성자를 호출할 수 있다. 이것을 방어하려면, 생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.

* 첫번째 코드 장점
  * 해당 클래스가 싱글턴임이 API에 명백히 드러난다. \(public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.\)
  * 간결함
* 두번째 코드 장점 
  * API 를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
  * 원한다면 정적 팩터리를 제네릭 싱턴 팩터리로 만들 수 있다.
  * 정적 팩터리의 메서드 참조를 공급자\(supplier\)로 사용할 수 있다. Elvis::getInstance를 Supplier&lt;Elvis&gt;로 사용히는 식이다.
* 세번째 코드 장점 
  * public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.
  * 대부분 상황에서는 원소가 하나뿐인 열거 타입이 **싱글턴을 만드는 가장 좋은 방법**이다.

싱글턴 클래스를 직렬화하려면, 단순히 Serializable 구현만으로는 부족하다. 모든 인스턴스 필드를 transient이라고 선언하고 readResolve 메서드를 제공해야 한다\(아이템 89\). 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때 마다 새로운 인스턴스가 만들어진다.

## 아이템 04: 인스턴스화를 막으려거든 private 생성자를 사용하라.

정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다. \(예, java.util.Arrays, java.lang.Math\) 또한，java.util.Collections처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아놓을 수도 있다.

생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 사용자는 자동 생성된 것인지 구분할 수 없다.

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. 하위 클래스를 만들어 인스턴스화하면 그만이다.

```java
public class UtilityClass {
    // Suppress default constructor for noninstantiability -> 사용자를 위해 이렇게 작성해야...
    private UtilityClass() {
       throw new AssertionError();
    }
    ... // Remainder omitted
}
```

상속을 불가능하게 하는 효과도 있다. 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 private으로 선언했으니 생성자에 접근할 길이 막혀버린다.

## 아이템 05: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

사용하는 자원에 따라 동작이 달라지는 클래스에는 **정적 유틸리티** 클래스나 **싱글턴** 방식이 적합하지 않다.

```java
// 나쁜 예 1
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // Noninstantiable
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
        
// 나쁜 예 2 
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}   
​
// 두 방식 모두 사전을 단 하나만 사용한다고 가정한다..
---------------------------------------------------------------
// 개선한다면...
// Spellchecker가 여러 자원 인스턴스를 지원하기 위해,
// 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이다.
​
// 좋은 예 - 의존 객체 주입 패턴
// Dependency injection provides flexibility and testability
public class SpellChecker { // 사용하는 자원(dictionary)에 따라 동작이 달라지는 클래스
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) { // 의존객체인 사전 주입
      this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

위코드는 의존 객체 주입 패턴이라 불리는데, 불변\(아이템 17\)을 보장하여 \(같은 자원을 사용하려는\)여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다.

쓸만한 변형으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.\(=Factory Method pattern\)

완벽한 예로 Supplier 인터페이스가 있다. Supplier를 입력으로 받는 메서드는 한정적 와일드카드 타입\(bounded wildcard type, 아이템 31\)을 사용해 매개변수를 제한해야 한다. 클라이언트는 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있다. 비록 의존성이 수 천 개 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 하지만, 의존 객체 주입 프레임워크\(대거, 주스, 스프링\)를 사용하면 이런 어질러짐을 해소할 수 있다.

## 아이템 06: 불필요한 객체 생성을 피하라.

```java
// 실행될 때마다 String 인스턴스를 새로 만든다
String s = new String("bikini"); // DON'T DO THIS! 
​
String s = "bikini"; // 이렇게 사용
```

정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. \(아이템 1\) `Boolean.valueOf(String)`

```java
// 숫자인지를 확인하는 메서드 작성
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}   
```

이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신\(finite state machine\)을 만들기 때문에 인스턴스 생성 비용이 높다.

```java
// 재사용이 많을 경우 좋은 예
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
                            "^(?=.)M*(C[MD]|D?C{0,3})"
                            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
      return ROMAN.matcher(s).matches();
    }
}
```

저자 컴퓨터에서 길이가 8인 문자열을 입력했을 때 6.5초 빨라진 것을 확인했다.

\(주제 어긋?\) 객체가 불변이라면 재사용해도 안전함이 명백하다. 예컨대 Map 인터페이스의 keyset 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰를 반환한다. keyset을 호출할 때마다 새로운 Set 인스턴스가 만들어지리라고 순진하게 생각할 수도 있지만 사실은 매번 같은 Set 인스턴스를 반환할 지도 모른다. 하나를 수정하면 다른 모든 객체가 따라서 바뀐다. 모두가 똑같은 Map 인스턴스를 대변하기 때문이다.

불필요한 객체를 만들어내는 또 다른 예로 오토박싱을 들 수 있다. 오토박싱 코드는 의미상으로는 별다를 것 없지만 성능에서는 그렇지 않다. 오토박싱이 숨어들지 않도록 주의하자.

```java
// 모든 양의 정수의 총합
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i
    return sum; 
}
​
// Long 인스턴스가 약 231개나 만들어진다. - 아마 gc가 리하고 남은 객?
// long으로만 바꿔주니 저자 컴퓨터에서 6.3초에서 0.59초로 빨라졌다.
```

단순히 객체 생성을 피하고자 본인 만의 객체 풀을 만들지는 말자. 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 풀을 만들어 재사용하는 편이 낫다. 하지만 일반적으로는 풀은 코드를 헷갈리게 만들고 메모리 사용량과 성능에 영향을 준다.

이번 아이템은 **방어적 복사\(defensive copy\)**를 다루는 아이템 50과 대조적이다.   
이번 아이템\(6\)이 **기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라**라면, 아이템 50은 **새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라**다.

**방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해**가, 필요 없는 객체를 반복 생성했을 때의 피해보다 **훨씬 크다는 사실**을 기억하자. 방어적 복사에 실패하면 언제 터져 나올지 모르는 **버그와 보안구멍으로 이어지지만** **불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다.**


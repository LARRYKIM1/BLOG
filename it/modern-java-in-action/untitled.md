---
description: 'Raoul-Gabriel Urma 저, 우정은 역  「Modern Java in Action, 2019」를 읽고 정리하였습니다.'
---

# \# 4장스트림 소개

#### 주제

* 스트림이란무엇인가?
* 컬렉션과스트림
* 내부반복과외부반복
* 중간연산과최종연산

어떤 사람은 컬렉션에서 칼로리가 적은 요리만 골라 특별 건강 메뉴를 구성하고 싶을지도 모른다. 이때 컬렉션으로 데이터를 그룹화하고 처리할 수 있다.

데이터베이스는 선언형이므로, SQL 질의 언어 우리가 기대하는 것이 무엇인지 직간단히표현할 수 있다.

```sql
 SELECT name FROM dishes WHERE calorie < 400
```

컬렉션으로도 이와 비슷한 기능을 만들 수 있지 않을까?

많은 요소를 포함하는 커다란 컬렉션은 어떻게 처리해야 할까? 성능을 높이려면 멀티코어 아키텍처를 활용해서 ****병렬로 컬렉션의 요소를 처리해야 한다. 하지만 코드를 구현하는 것은 단순 반복 처리 코드에 비해 복잡하고 디버깅도 어렵다.

자바 언어 설계자들은 어떤 결정을 내렸을까? 답은 스트림이다.

## 4.1 스트림이란무엇인가?

* 스트림을 이용하면 선언형으로\(질의로 표현\) 컬렉션 데이터를 처리할 수 있다.
* 데이터 컬렉션 반복을 멋지게 처리하는 기능
* 데이터를 투명하게 병렬로 처리할 수 있다. \(7장 참고\)

저칼로리의 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 **자바 7** 코드와 **자바 8** 코드 비교.

```java
// 메뉴 데이터
List<Dish> menu = Arrays.asList(
      new Dish("pork", false, 800, Dish.Type.MEAT),
      new Dish("beef", false, 700, Dish.Type.MEAT),
      new Dish("chicken", false, 400, Dish.Type.MEAT),
      new Dish("french fries", true, 530, Dish.Type.OTHER),
      new Dish("rice", true, 350, Dish.Type.OTHER),
      new Dish("season fruit", true, 120, Dish.Type.OTHER),
      new Dish("pizza", true, 550, Dish.Type.OTHER),
      new Dish("prawns", false, 400, Dish.Type.FISH),
      new Dish("salmon", false, 450, Dish.Type.FISH)
  );
​
// 저칼로리의 요리
List<Dish> lowCaloricDishes = new ArrayList<>(); // 가비지 변수
for(Dish dish: menu) {
    if(dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}
// 칼로리 순으로 정렬
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});
// 정렬된 리스트 이름만 리스트에 넣기
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish: lowCaloricDishes) {    
    lowCaloricDishesName.add(dish.getName());
}
----------------------------------------------------------------
// 자바 8 버전
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
​
List<String> lowCaloricDishesName =
                menu.stream()
                    .filter(d -> d.getCalories() < 400) // 선택
                    .sorted(comparing(Dish::getCalories)) // 정렬
                    .map(Dish::getName) // 추출
                    .collect(toList());  // 저장
​
```

 lowCaloricDishes라는 **가비지 변수**를 사용했다. \(= 컨테이너 역할만 하는 중간 변수\)

`menu.`**`parallelStream()`**`.filter(~`은 병렬로 실행된다.

소프트웨어공학적으로 스트림의 새로운 기능이 주는 다양한 이득을 보자.

* 선언형으로 코드
  * 루프와 if 조건문 등의 제어 블록을 사용해서 어떻게\(**How**\) 동작을 구현할지 지정할 필요 없다. **What**만 지정.
* 여러 빌딩 블록 연산\(filter 등\)을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있다.

filter \(또는 sorted, map, collect\) 같은 연산은 **고수준 빌딩 블록\(High-level building block\)**으로 이루어져 있으므로 특정 **스레딩 모델에 제한되지 않고** 자유롭게 어떤 상황에서든 사용할 수 있다. \(= 스레드와 락을 걱정할 필요 x\)

#### 스트림 장점 요약

* 선언형
* 조립할 수 있음
* 병렬화

## 4.2 스트림 시작하기

스트림이란 정확히 뭘까? 스트림이란 **데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**로 정의할 수 있다.

* 연속된 요소
  * 컬렉션은 자료구조이므로 컬렉션에서는 시간과공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다.\(ArrayList, LinkedList 어느것 사용할 것인지 같은..\) 반면 스트림은 표현 계산식이 주를 이룬다. 즉, **컬렉션**의 주제는 **데이터**고 **스트림**의 주제는 **계산**이다
* 소스
  * 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다.
* 데이터 처리 연산
  * 순차적으로 또는 병렬로 실행할 수 있다.
* 파이프라이닝 Pipelining : 스트림 자신을 반환한다. 그 덕분에 laziness, short-circuiting 같은 최적화도 얻을 수 있다.

## 4.3 컬렉션과 스트림의 차이

* 데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이다.
* 컬렉션의 모든 요소는 컬렉션에 추가하 기 전에 계산되어야 한다.
* 반면 스트림은 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조다.

스트림은 딱 한번만 탐색할 수 있다. \(Traversable only once\)

```java
List<String> title = Arrays.asList("Modern", "Java", "In", "Action");
Stream<String> s = title.stream();
        s.forEach(System.out::println);
        s.forEach(System.out::println); // IllegalStateException 에러 
```

* 컬렉션은 **외부 반복**, 스트림은 **내부 반복**
* for-each를 이용하는 외부 반복에서는 병렬성을 스스로 관리해야 한다.\(synchronized로 시작하는 긴 전쟁의 서막..\)
* 스트림의 반복을 숨겨주는 대부분의 연산은 \(filter, map\) 람다 표현식을 인수로 받으므로 3장에서 배운 동작 파라미터화를 활용할 수 있다.

### 퀴즈

다음 외부 반복을 참고해서 스트림 동작을 사용해 리팩터링 해보자.

```java
List<String> highCaloricDishes = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while(iterator.hasNext()) {
    Dish dish = iterator.next();
    if(dish.getCalories() > 300) {
        highCaloricDishes.add(d.getName());
    }
}
```

## 4.4 스트림 연산

스트림 인터페이스의 연산을 크게 두 가지로 구분한다. \(중간연산, 최종연산\)

* 중간 연산은 다른 스트림을 반환한다. 따라서 연결해서 질의를 만들 수 있다. \(빌더 패턴과 비슷\)
* 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다. \(게으름\)
* 게으른 특성 덕분에 몇 가지 최적화 효과를 얻을 수 있었다. 코드로 확인해보자.

```java
//첫째: 300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택했다. (limit 연산 그리고 쇼트서킷 기법)
//둘째: filter와 map은 서로 다른 연산이지만 한과정으로 병합되었다. (루프 퓨전 loop fusion 기법)
List<String> names =
    menu.stream()
    .filter(dish -> {
        System.out.println("filtering:" + dish.getName());
        return dish.getCalories() > 300;
    })
    .map(dish -> {
        System.out.println("mapping:" + dish.getName());
        return dish.getName();
    })
    .limit(3)
    .collect(toList());
System.out.println(names); 
```


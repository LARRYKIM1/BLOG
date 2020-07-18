# 실습

### &lt;실습 계획&gt;

```text
1. 영화를 할인조건에 따라 2개 만든다. Movie ( 이름, 러닝타임, 요금, 할인조건리스트(퍼센트,특정금액) )
MovieType에 따라 다르게 생성하기 위해 생성자 3개가 있다.
멤버 변수로 총 7개가 있다.
    private String title;
    private Duration duration;
    private Money fee;
    private List<DiscountCondition> discountCondition;

    private MovieType movieType;
    private int discountAmount;
    private double discountPercent;

2. 할인 조건을 2종류 만든다. DiscountCondition. DiscountConditionType 확인.
타입에는 2종류(시퀀스, 시간대)가 있고, 조건에는 타입, 순번, 요일, 시작시간, 종료시간이 멤버 변수로 있다.

3. 상영정보를 만든다. Screening ( 영화, 순번, 상영시간  )
4. 판매업체와 판매업체가 만들어줄 예약을 만든다. ReservationAgency, Reservation
예약에는 관람객, 상영영화=볼영화, 할인 적용된 가격, 관라객수를 프린트한다.

5. 고객을 만든다. Customer
6.  볼영화, 고객, 관람객수를 넣어 ReservationAgency의 reserve()메서드를 호출한다. 예약이 된다.


6. 고객, 볼영화, 지불금액, 관람객수를 넣어 예약을 만든다. Reservation
7. 볼영화, 고객, 관람객수를 넣어 ReservationAgency의 reserve()메서드를 호출한다. 예약이 된다.

8. Screening에서 getMovieType으로 어떤 할인조건이 있는지 확인후 영화에게 가격계산하라는 메시지 전송
9. Movie 영화에 들어갈 메서드 calculateAmountDiscountedFee/calculatePercentDiscountedFee/calculateNoneDiscountedFee/isDiscountable 4개 추가.
10. isDiscountable 만들기
```

### &lt;Step02 메인&gt;

```java
import com.larrykim.chap04.money.Money;
import java.time.DayOfWeek;
import java.time.Duration;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.ArrayList;
import java.util.List;

public class Step02AloneMain {
    public static void main(String[] args) {
        List<Movie> movieList = generateMovie(); // 영화 3개 생성
        List<Screening> screeningList = generateScreening(movieList); // 영화 상영 정보 생성 각 3개씩 총 9개

//        Customer customer = new Customer("김유철", "id1234");

        List<Customer> customerList = generateCustomer();
        ReservationAgency reservationAgency = new ReservationAgency();
        for (int i = 0; i < 9; i++) {
            Reservation reservation = reservationAgency.reserve(screeningList.get(i), customerList.get(i), 2);

            // 0.1 할인비율로 적용된 소리꾼을 예약한 고객은 총금액이 18000원
            // 나머지는 800원이 할인되서 나온다. (살아있다, 온워드)
            System.out.printf("============= %d번째 예매 내역 ==============\n", i + 1);
            System.out.println(reservation.toString());
            System.out.println("==========================================");
        }

    }

    private static List<Customer> generateCustomer() {
        // 상영하는 영화 3개가 각 3번 9번 상영하므로, 한명당 한 영화 예약을 위해 9명 고객 생성
        List<Customer> customerList = new ArrayList<>();
        for (int i = 0; i <= 9; i++) {
            Customer customer = new Customer("고객" + (i + 1), "id" + (i + 1));
            customerList.add(customer);
        }
        return customerList;
    }

    private static List<Screening> generateScreening(List<Movie> movieList) {
//        for (Movie movie : movieList) {
//            System.out.println(movie);
//        }

        LocalDateTime localDateTime = LocalDateTime.of(2020, 7, 6, 9, 10);

        List<Screening> screeningList = new ArrayList<>();

//        Screening screening1 = new Screening(movieList.get(0), 0, localDateTime);
//        screeningList.add(screening1);
//        Screening screening2 = new Screening(movieList.get(1), 1, localDateTime);
//        screeningList.add(screening2);
//        Screening screening3 = new Screening(movieList.get(2), 2, localDateTime);
//        screeningList.add(screening3);

        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                // 모든 영화가 3번 상영 되도록 총 9개 생성
                Screening screening = new Screening(movieList.get(j), i + 1, localDateTime);
                screeningList.add(screening);
            }
        }

//        for (Screening screening : screeningList) {
//            Debug.println("상영정보 \n",screening.toString());
//        }

        return screeningList;
    }

    private static List<Movie> generateMovie() {
        // 1번째 영화에 할인 조건 추가 - SEQUENCE
        DiscountCondition discountConditionSequence = new DiscountCondition(1);
        // 월요일 아침 9시~12시 할인 조건 추가 - PERIOD
        DiscountCondition discountConditionPeriod = new DiscountCondition(DayOfWeek.MONDAY, LocalTime.of(9, 00), LocalTime.of(12, 00));

        List<Movie> movieList = new ArrayList<>();

        // 할인 조건 다른 3종류 영화 생성 - 모두 9:00~12:00 조조 할인과 1번째 영화 할인
        Movie movie1 = new Movie("살아있다", Duration.ofHours(2), Money.wons(10000), 800, discountConditionSequence, discountConditionPeriod);
        Movie movie2 = new Movie("소리꾼", Duration.ofHours(2), Money.wons(10000), 0.1, discountConditionSequence, discountConditionPeriod);
        Movie movie3 = new Movie("온워드", Duration.ofHours(2), Money.wons(10000), discountConditionSequence, discountConditionPeriod);

        movieList.add(movie1);
        movieList.add(movie2);
        movieList.add(movie3);

        return movieList;
    }
}

```


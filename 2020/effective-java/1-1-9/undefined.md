# 발표 코드

```java
package com.larrykim.Test01;

import java.math.BigInteger;
import java.time.Instant;
import java.util.*;

import static java.util.Date.*;

public class Member {
    private String name;
    private String address;

    public static final Member MEMBER = new Member("김유철");

    public Member() {
    }

    public Member(String name) {
        this.name = name;
    }

    public static Member withName(String name) {
        return new Member(name);
    }

    public static Member withAddress(String address) {
        Member member = new Member();
        member.setAddress(address);
        return member;
    }

    public static Member getInstance() {
        return MEMBER;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    enum days {
        SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
    }

    public static void main(String[] args) {
        Member member1 = new Member("강남");
        Member member2 = Member.withName("강남");
        Member member3 = Member.withAddress("강남구");

        Member member5 = Member.getInstance();
        Member member6 = Member.getInstance();
        Member member7 = Member.getInstance();
        Member member8 = Member.getInstance();

        Instant instant = Instant.now();
        Date date = Date.from(instant);

        Set<days> set = EnumSet.of(days.TUESDAY, days.WEDNESDAY);
        Iterator<days> iter = set.iterator();

        BigInteger bi1 = new BigInteger(10, new Random());
        BigInteger bi2 = BigInteger.probablePrime(10, new Random());


        List<Integer> integerList=new ArrayList<>();
        integerList.add(1);
        integerList.add(2);
        integerList.add(3);

        List<Integer> unmodifiableList = Collections.unmodifiableList(integerList);
        System.out.println(unmodifiableList);



//        Collection syncedCollection = Collections.synchronizedCollection(originalCollection);
//        Set syncedSet = Collections.synchronizedSet(new HashSet());
//        List<Integer> unmodifiableList = Collections.unmodifiableList(integerList);
//        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(originalMap);

    }

    @Override
    public String toString() {
        return "Member{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```


---
description: '조영호 저 "오브젝트: 코드로 이해하는 객체지향 설계, 2019"를 읽고 정리한 내용입니다.'
---

# \# 11장



### 합성과 유연한 설계

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

### 기본정책과 부가정책 조합하기





### 상속을 이용해서 기본정책 구현하기

### 기본정책에 세금정책 조합하기

> 훅메서드란?
>
> 추상메서드와 동일하게 자식 클래스에서 오버라이딩할 의도로 메서드를 추가했지만 편의를 위해 기본구현을 제공하는 메서드


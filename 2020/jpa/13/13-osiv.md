# \#13 OSIV 실습

{% embed url="https://github.com/Incheol-Jung/jpa-repository-test" %}

스터디원 인철님의  기텁을 보고 실행하였습니다. 

## 디렉토리 구조

![&#xB514;&#xB809;&#xD1A0;&#xB9AC;&#xAD6C;&#xC870;](../../../.gitbook/assets/image%20%2869%29.png)

## 1 person.http 실행

테이블은 Person과 Item이 생성된다. \(@ManyToOne-지연로딩으로 변경\)

name01과 name02를 만들고 각 이름에 아이템 10개씩 넣어진다.

![](../../../.gitbook/assets/image%20%2871%29.png)

![](../../../.gitbook/assets/image%20%2870%29.png)

### 실행코드는 다음 순으로 간다.

```java
@RestController
@RequestMapping(value = "/persons", produces = "application/json")
public class PersonController {

    @Autowired
    PersonService personService;

    @GetMapping(value = "")
    public List<Person> get() {
        return personService.get();
    }

    @PostMapping(value = "")
    public Person get(String name) {
        return personService.save(name);
    }
}
---------------------------------------------------------------------
@Service
public class PersonService {
    @Autowired
    PersonRepository personRepository;

    @Autowired
    ItemRepository itemRepository;

    public List<Person> get() {
        return personRepository.findAll();
    }

    public Person save(String name) {
        Person person = new Person(name);
        person = personRepository.save(person);

        List<Item> items = new ArrayList<>();
        for(int i=0; i<10; i++){
            Item item = new Item("item" + i);
            item.setPerson(person);
            items.add(item);
        }
        itemRepository.saveAll(items);

        return person;
    }
}
---------------------------------------------------------------------
// Spring Jpa Data 사용중
public interface PersonRepository extends JpaRepository<Person, Integer> {
}

---------------------------------------------------------------------
```

### .http 실행으로 로그 생성 확인

http 상태코드 200이 나오며 DB에 잘 들어갔다.

![](../../../.gitbook/assets/image%20%2867%29.png)

## 2 Item.http 실행

![](../../../.gitbook/assets/image%20%2868%29.png)

Stream을 사용해 모든 Person이 Json으로 넘어온다.

### 실행코드는 다음 순으로 간다.

```java
@RestController
@RequestMapping(value = "/items", produces = "application/json")
public class ItemController {

    @Autowired
    ItemService itemService;

    @GetMapping(value = "")
    public List<ItemResponse> get() {
        List<Item> items =  itemService.get();
        Item firstItem = items.get(0);

        return items.stream()
                    .map(item -> new ItemResponse(
                            item.getId(), item.getItemName(), item.getPerson()))
                    .collect(Collectors.toList());
    }

}
------------------------------------------------------------------
@Service
public class ItemService {
    @Autowired
    ItemRepository itemRepository;

    @Transactional
    public List<Item> get() {
        List<Item> items = itemRepository.findAll();

        return items;
    }
}
```

### 넘어온 Json 

name01 아이템 10개, name02 아이템 10개

```java
[
  {
    "id": 1,
    "itemName": "item0",
    "person": {
      "id": 1,
      "name": "name01",
      "hibernateLazyInitializer": {}
    }
  },
 // 중간 생략
 ,
  {
    "id": 20,
    "itemName": "item9",
    "person": {
      "id": 2,
      "name": "name02",
      "hibernateLazyInitializer": {}
    }
  }
]
```

이제 기초공사가 완료되었다.   
JSP 뷰에서 프록시를 초기화 할 수 있는지 확인해보자.

## 3 view.http 실행 - JSP 프록시 실험 \(OSIV 범위 테스트\)

![](../../../.gitbook/assets/image%20%2865%29.png)

### jsp 코드

```java
<c:forEach var="item" items="${items}" varStatus="items">
    <p>${item.id} / <c:out value="${item.itemName}" /> / <c:out value="${item.person.name}" /> </p>
</c:forEach>
```

### 실행코드는 다음 순으로 간다.

```java
@Controller
public class ViewController {
    @Autowired
    ItemService itemService;

    @RequestMapping("/")
    public ModelAndView main() {
        ModelAndView mv = new ModelAndView();
        List<Item> items = itemService.get();
        
        mv.addObject("items", items);
        mv.setViewName("index");
        
        return mv;
    }
}
index.jsp --------------------------------------------------- 
<c:forEach var="item" items="${items}" varStatus="items">
    <p>${item.id} / <c:out value="${item.itemName}" /> / <c:out value="${item.person.name}" /> </p>
</c:forEach>
```

### html로 변환된 jsp 결과 

**`<c:out value="${item.person.name}"/>`**jsp에서 프록시가 초기화 되엇다.

```markup
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
    <title>Jpa Repository Page</title>
</head>
<body>

    <p>1 / item0 / name01 </p>
    
    중간 생략...
    
    <p>20 / item9 / name02 </p>

</body>
</html>
```






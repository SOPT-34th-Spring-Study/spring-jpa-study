# 6️⃣ 주문 도메인 개발
## 주문, 주문상품 엔티티 개발

## Order 엔티티

### 가변 인자(Varargs)

```java
    //==생성 메서드==//
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }
```

메서드의 매개변수에 있는 `OrderItem... orderItems` 는 가변 인자를 의미한다. 이 때, *orderItems 라는 OrderItem 형 배열이 선언*되고 메서드가 호출될 때 넘겨받은 인자들이 이 배열로 묶어 처리된다.

**❓가변인자를 사용한 이유는 무엇일까❓**

아직 프로그램 상에서 이 메서드를 사용하는 코드는 없지만, 아래와 같이 createOrder를 호출 할 때 여러 개의 OrderItem을 전달할 수 있다.

```java
OrderItem item1 = new OrderItem();
OrderItem item2 = new OrderItem();
OrderItem item3 = new OrderItem();

Order order = Order.createOrder(member, delivery, **item1, item2, item3**);
```

이렇게 되면 OrderItem[ ]배열로 처리되어 orderItems는 {”item1”, “item2”, “item3”}이 된다.

**for-each**

```java
for (OrderItem orderItem : orderItems) {
    order.addOrderItem(orderItem);
}
```

orderItems 배열의 각 요소를 하나씩 순회하면서 orderItem 변수에 할당한 다음, 작업을 수행한다.

---

### 💡  단축키 꿀 팁

단축키 `option+enter` 과 `option+command+N` 을 순차로 사용하면 아래와 같은 형태를 가독성 좋게 자동으로 바꿔준다!

```java
 		//==조회 로직==//
    public int getTotalPrice() {
        int totalPrice = 0;
        for ( OrderItem orderItem : orderItems){
            totalPrice += orderItem.getTotalPrice();
        }
        return totalPrice;
    }
```

```java
    public int getTotalPrice() {
        return orderItems.stream()
                .mapToInt(OrderItem::getTotalPrice)
                .sum();
    }
```

## 주문 리포지토리 개발

큰 내용 없음.

## 주문 서비스 개발

## CascadeType

```java
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {

        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);
        Order order = Order.createOrder(member, delivery, orderItem);

        //주문 저장
        orderRepository.save(order);
        return order.getId();
    }
```

**❓ save는 order만 해도 괜찮은건가요? orderItem과 delivery는요? ❓**

Order 엔티티를 살펴보면 orderItems과 delivery에 대해서 `CascaseType.All`이 걸려 있는 것을 확인할 수 있다. 이를 통해 *order 하나만 저장(상태 변화)해도 나머지 둘도 자동으로 persist*가 된다.

```java
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
```

### CascadeType의 사용 범위

 라이프사이클 내에서 동일하게 관리되고, OrderItem과 Delivery를 Order만 사용하는 private owner인 경우이므로 CascadeType.All을 걸었다. 

 하지만, 다른 엔티티에서도 Delivery를 많이 가져다 쓰는 경우에는 All을 쓰는 것은 위험하다. 이 경우는 별도의 repository를 만들어서 독립적으로 관리하는 것이 좋다.

---

## `@NoArgsConstructor(access = AccessLevel.*PROTECTED*)`

```java
        //주문상품 생성 
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);
        
        //주문상품 생성 (좋지 않은 예)
        OrderItem orderItem1 = new OrderItem();
        orderItem1.setCount(1000);
```

### 정적 팩토리 메서드 사용을 보장하자

**정적 팩토리 메서드(Static Factory Method)란,** 

객체의 생성을 담당하는 클래스 메서드로써, new 를 직접적으로 사용하지 않을 뿐, *클래스 내에 선언되어 있는 메서드를 내부의 new로 간접 이용해서 객체를 생성한 다음 반환*하는 것이다. → 기본 생성자를 외부에서 호출할 수 없게 해서 객체의 일관성을 유지시킬 수 있다.

**방법 1**

엔티티 클래스 내에 기본 생성자의 접근 제어자를 protected로 둔다.

```java
  protected OrderItem() { }
```

⭐ **방법 2** ⭐ 

엔티티 클래스의 위에 **`@NoArgsConstructor(access = AccessLevel.PROTECTED)`** 어노테이션을 붙여준다.



## 주문 기능 테스트

## 유용한 단축키

### **`option + command + M`**

자주 쓰이는 코드를 외부 함수로 따로 빼준다.

```java
    private Book createBook() {
        Book book = new Book();
        book.setName("시골 JPA");
        book.setPrice(10000);
        book.setStockQuantity(10);
        em.persist(book);
        return book;
    }
```

```java
 Book book = createBook();
```

### **`option + command + p`**

지역변수를 매소드 파라미터로 추출해준다.
![image](https://github.com/user-attachments/assets/ef774837-e7af-4134-b093-c676e1a2a171)


### `command + shift + T`

Test를 생성하기도 하지만, Test 생성 후에는 Test인 클래스와 아닌 클래스를 왔다갔다 바로 할 수 있는 단축키

## 주문 검색 기능 개발

![image](https://github.com/user-attachments/assets/d5d88598-f858-42d0-87da-eb9ed6c5bbf6)


회원 이름과 주문 상태(주문, 취소)를 무엇을 선택하느냐에 따라서 필터링 해서 해당하는 주문 상품들을 보여주는 쿼리문을 작성했다. 

```java
    public List<Order> findAll(OrderSearch orderSearch) {

        return em.createQuery("select o from Order o join o.member m " +
                "where o.status = :status " +
                "and m.name like :name", Order.class)
                .setParameter("status", orderSearch.getMemberName())
                .setMaxResults(1000) //최대 1000건
                .getResultList();
    }
```

하지만, 멤버 이름과 주문 상태가 따로 선택되어 있지 않은 경우에는 아래 쿼리처럼 모든 주문 상품들을 다 가져와야 한다.. 

```java
    public List<Order> findAll(OrderSearch orderSearch) {

        return em.createQuery("select o from Order o join o.member m ", Order.class)
                .setMaxResults(1000) //최대 1000건
                .getResultList();
    }
```

1. 사용자가 파라미터를 넣지 않은 경우 `(null, null)`
2. 사용자가 회원 이름으로 검색한 경우 `(”회원 1”, null)`
3. 사용자가 회원 이름과 주문 상태로 검색한 경우 `(”회원 1”, CANCEL)`

→ 이렇게 실행 시점에 쿼리의 일부가 변경될 수 있는 **동적 쿼리**를 JPA에선 어떻게 해결할 수 있을까?

## QueryDSL

강의에서는 JPQL과 JPA Criteria를 소개하고, QueryDSL은 뒤에서 살펴본다고 하셨지만 저는 그냥 바로 QueryDSL를 사용해보겠습니다! (왜냐면 굳이 안쓸것들을 공부하긴 귀찮?음…….)

Builder Pattern 스타일로 동적 쿼리를 작성할 수 있어 가독성과 객체지향성을 높일 수 있습니다.

### QueryDSL 사용 방법

1. **Dependencies 추가 ( SpringBoot 3.x )**

```java
	// queryDSL
	implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
	annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```

1. **Q-class 생성**

```java
./gradlew clean build
```

[build] - [generated] 폴더 안에 Q-class들이 생성된 것을 확인할 수 있다.
![image](https://github.com/user-attachments/assets/73be5879-32d8-4502-a697-6984a0a994b1)


2. **QueryDSL Configuration**

프로젝트에서 QueryDSL을 사용하기 위해선 QueryDSL 설정이 필요한데, 아래와 같이 JPAQueryFactory를 Bean으로 등록한다.

```java
@Configuration
@RequiredArgsConstructor
public class QueryDSLConfig {

    private final EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```

3. **Repository를 통해서 QueryDSL 사용**

기존에 있던 Repository의 CustomRepository 인터페이스를 정의하고 해당 인터페이스를 구현한 클래스를 기존 Repository에 상속해서 사용하는 방법이 있지만, 지금은 그냥 기존의 Repository에 바로 쿼리문을 작성하는 방법을 사용하겠다. 관심이 있다면 아래 블로그를 참고!

[[Gradle] SpringBoot 3.x + QueryDSL 적용하기](https://velog.io/@kimsundae/Gradle-SpringBoot-3.x-QueryDSL-적용하기)

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

   '''
   
    private final JPAQueryFactory jpaQueryFactory;

    @Transactional
    public List<Order> findAll(OrderSearch orderSearch) {
        QOrder order = QOrder.order;
        QMember member = QMember.member;

        BooleanBuilder builder = new BooleanBuilder();

        // 동적 쿼리 조건 추가
        if (orderSearch.getOrderStatus() != null) {
            builder.and(order.status.eq(orderSearch.getOrderStatus()));
        }
        if (orderSearch.getMemberName() != null && !orderSearch.getMemberName().isEmpty()) {
            builder.and(member.name.like("%" + orderSearch.getMemberName() + "%"));
        }

        return jpaQueryFactory
                .selectFrom(order)
                .join(order.member, member)
                .where(builder)
                .limit(1000)
                .fetch(); //쿼리 실행해서 결과를 list로 반환
    }
    
    '''
}
```

- **BooleanBuilder**

동적 쿼리의 조건을 조립하는 데 사용된다. 조건을 추가할 때 `and`나 `or` 메서드를 사용하여 조건을 결합할 수 있다. 이를 사용하여 조건을 동적으로 추가하거나 제거할 수 있다.

- **% 문자**

 `LIKE` 연산자에서 사용되는 와일드카드 문자로써 `%`는 0개 이상의 문자와 일치함을 의미한다. 따라서 `%`가 양쪽에 있는 것은 주어진 문자열이 `name`의 어디에 위치하든지 상관없이 포함되면 조건이 참이 되도록 한다. 

 예를 들어 `orderSearch.getMemberName()`이 `"John"`인 경우, `member.name` 필드가 `"John"`을 포함하는 모든 레코드를 검색하게 된다. `name` 필드가 `"John Doe"`인 경우도 조건에 맞게 된다.

 ---

 # 7️⃣ 웹 계층 개발

 ## 홈 화면과 레이아웃

## **뷰 템플릿 변경사항을 서버 재시작 없이 즉시 반영하기**

1. builde.gradle에 spring-boot-devtools 의존성 추가
    
    ```java
    implementation 'org.springframework.boot:spring-boot-devtools'
    ```
    
2. html 파일 수정 후 build-> Recompile 하면 바로 반영된 것 확인 가능! `command +shitft + F9`

## 부트스트랩

예쁜 디자인을 위해서~ 

1. https://getbootstrap.com/ 에 들어가서 최신버전 다운로드 받는다.
2. 아래와 같은 파일이 다운로드 되는데 이 두 파일을 [resources] - [static]에 그대로 붙여넣는다.

![image](https://github.com/user-attachments/assets/f3dc1320-6f1b-4295-96cd-8acda49cf25a)


![image](https://github.com/user-attachments/assets/2b9d6919-20cd-4168-b98d-882a264a6a6d)


1. resources/static/css/jumbotron-narrow.css 추가해서 코드 넣으면 더 예쁜 디자인이 보인다.

## 회원 등록

## 유효성 검사( @Valid )

spring boot version이 **2.3** 이상일 경우, 아래 의존성을 추가해 주어야 사용할 수 있다.

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

- 유효성 검사에서 사용할 수 있는 어노테이션
    
    ```java
    @Null  // null만 혀용합니다.
    @NotNull  // null을 허용하지 않습니다. "", " "는 허용합니다.
    @NotEmpty  // null, ""을 또는 리스트 [] 빈값 허용하지 않습니다. " "는 허용합니다.
    @NotBlank  // null, "", " " 모두 허용하지 않습니다.
    
    @Email  // 이메일 형식을 검사합니다. 다만 ""의 경우를 통과 시킵니다. @Email 보다 아래 나올 @Patten을 통한 정규식 검사를 더 많이 사용합니다.
    @Pattern(regexp = )  // 정규식을 검사할 때 사용됩니다.
    @Size(min=, max=)  // 문자길이를 제한할 때 사용, int는 불가!
    
    @Max(value = )  // 숫자 value 이하의 값을 받을 때 사용됩니다.
    @Min(value = )  // 숫자 value 이상의 값을 받을 때 사용됩니다.
    
    @Pattern(regexp = )		// 정규표현식으로 검증식 세울 수 있습니다.
    
    @Positive  // 값을 양수로 제한합니다.
    @PositiveOrZero  // 값을 양수와 0만 가능하도록 제한합니다.
    
    @Negative  // 값을 음수로 제한합니다.
    @NegativeOrZero  // 값을 음수와 0만 가능하도록 제한합니다.
    
    @Future  // 현재보다 미래
    @Past  // 현재보다 과거
    
    @AssertFalse  // false 여부, null은 체크하지 않습니다.
    @AssertTrue  // true 여부, null은 체크하지 않습니다.
    
    @Valid	// 해당 object validation 실행
    ```
    

## `@Valid` 와 `BindingResult`

**`@Valid`** 는 주로 스프링 MVC에서 `Request Body`를 검증하는 데 사용한다. DTO 클래스의 필드에 선언된 유효성 검사 어노테이션들을 기반으로 요청 데이터가 유효한지 검증한다. 만약 유효성 검사를 통과하지 못하면, **`BindingResult`** 객체에 오류가 저장된다.

MemberForm이라는 Requset DTO가 있다고 가정헤보자. 이 클래스의 name 필드에 `@NotEmpty` 로 유효성 검사 어노테이션을 적용하여서 name 은 빈 문자열이나 null 값을 가질 수 없다는 것을 명시해 주었다.

```java
@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수 입니다")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}
```

Controller 내의 회원 가입 함수(create)의 파라미터 부분에 `@Valid` 어노테이션을 적으면 유효성 검사를 할 수 있다. 동작 시, name필드에 null 값이 들어가면 발생한 오류를 `BindingResult` 객체에 저장하게 된다. 만약 유효성 검사에서 오류가 발생했다면, 다시 createMemberForm(회원 가입 페이지)로 리다이렉션 시킨다.

```java
    @PostMapping("/members/new")
    public String create(
            @Valid MemberForm form,
            BindingResult result
      ) {

        if (result.hasErrors()) {
            return "members/createMemberForm";
        }
     
     ...
   }
```

### BindingResult

`@Valid` 어노테이션과 함께 사용되어, 유효성 검사 결과를 담는 객체이다. 

## 변경 감지와 병합(merge)

## 준영속 상태

- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리 된 상태(영속성 컨텍스트가 더는 관리하지 않음)
- 영속성 컨텍스트가 제공하는 기능을 사용할 수 X

## 준영속 엔티티

준영속 엔티티는 JPA가 관리를 하지 않기 때문에, 객체 수정 시에도 DB에 Update가 일어나지 않는다.

- DB에 한 번 들어갔다가 나온 엔티티
- DB에 한 번 저장되어서 식별자(id)가 존재하는 객체

```java
    @PostMapping("/items/{itemId}/edit")
    public String updateItem(
            @ModelAttribute("form") BookForm form
    ) {
        Book book = new Book();

        book.setId(form.getId());
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/items";
    }
```

✔️ 위의 코드에서 book 객체는 `book.setId(form.getId(())` 함으로써, 이미 영속 엔티티였던 form의 id값을 book객체에 설정하게 되므로, book 객체는 *‘이미 DB에 저장된 식별자’*를 가지게 된다. 따라서 book은 준영속 엔티티인 것! 

→ 따라서 updateItem()에서 book 객체의 수정사항은 변경감지(Dirty Checking) 되지 않는다.

## 준영속 엔티티 수정 방법

### 1. **변경 감지 기능 사용 (Dirty Checking)**

영속성 컨텍스트에서 준영속 엔티티와 동일한 식별자를 가진 영속 상태의 엔티티를 다시 조회한 후에 해당 엔티티의 필드를 수정하는 방법이다. 

```java
@Transactional
void update(Item itemParam) {  // itemParam: 파라미터로 넘어온 준영속 상태의 엔티티
    Item findItem = em.find(Item.class, itemParam.getId()); // 같은 엔티티를 조회한다.
    findItem.setPrice(itemParam.getPrice()); // 데이터를 수정한다.
    // 트랜잭션 종료 시점에 변경 사항이 자동으로 반영된다. 
}
```

1) 영속성 컨텍스트에서 준영속 엔티티인 itemParam과 같은 id를 가진 영속 엔티티(findItem)를 찾아온다.

2) 영속 상태의 엔티티인 findItem의 필드를 수정한다.

3) 트랜젝션 커밋 시점에 내부적으로 flush()가 발생해서 변경 감지가 일어난다. → 따로 save 필요없음!

### 2. **병합 사용 (Merge)**

병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다.

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티 
			Item **mergeItem** = em.merge(item);
}
```

### 병합의 동작 방식
![image](https://github.com/user-attachments/assets/46320ca1-82e7-4381-9912-e8390533326c)


**병합 동작 방식**

1. merge(member) 를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
    
    2-1. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
    
3. 조회한 영속 엔티티( mergeMember )에 member 엔티티의 값을 채워 넣는다. (member 엔티티의 모든 값을 mergeMember에 밀어 넣는다. 이때 mergeMember의 “회원1”이라는 이름이 “회원명변경”으로 바
    
    뀐다.)
    
4. 영속 상태인 mergeMember를 반환한다.

강의의 코드로 merge를 실행하면 아래와 같은 역할을 하게 된다. 

```java
    @Transactional
    public **Item** updateItem(Long itemId, Book param) { 
        Item findItem = itemRepository.findOne(itemId); 
        findItem.setPrice(param.getPrice());
        ...
        **return findItem;**
    }
```

### 병합 사용 시 주의점

변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, **병합을 사용하면 모든 속성이 변경된다. 병합시 값이 없으면 null 로 업데이트 할 위험도 있다. (병합은 모든 필드를 교체한다.)**

예를 들어, item의 가격은 수정하지 않고 그대로 둔다(price = null )고 가정해보자. 병합을 사용하면 `em.merge(item)` 으로 파라미터에 객체를 통째로 넣어서 보내게 된다. 그러면 수정되어 반환된 객체의 price도 null로 수정되기 때문에 문제가 발생할 수 있다.

### **엔티티를 변경할 때는 항상 `변경 감지(Dirty Checking)`를 사용하자!**

merge를 사용하면 편하긴 하나, 단순한 경우에만 사용 가능.. 위험하니까 ㅜㅜ

- 컨트롤러에서 어설프게 엔티티 생성 금지
    
    ```java
        @PostMapping("/items/{itemId}/edit")
        public String updateItem(
               @PathVariable Long itemId,
               @ModelAttribute("form") BookForm form
        )
        {
            itemService.updateItem(itemId, form.getName(), form.getPrice(), form.getStockQuantity());
            return "redirect:/items";
        }
    ```
  

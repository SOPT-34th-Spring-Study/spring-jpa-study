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

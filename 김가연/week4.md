## 주문 도메인 개발

### 가변인자

가변인자, 말 그대로 여러 개의 매개변수를 받을 수 있다는 뜻이다. 가변인자를 사용하면 메서드 호출 시에 전달되는 인자의 개수를 동적으로 변경할 수 있다. 배열을 포함한 모든 참조자료형(Wrapper Class, String, Object, List, Map)이 가변인자로 사용 가능하지만 기본 자료형은 가변인자로서 사용할 수 없다.
가변인자를 사용하는 방법은 간단하다. 아래 코드처럼 변수 타입 뒤에 기호(...)를 붙여주면 된다. 다만, 다른 파라미터와 가변인자를 같이 사용하는 경우에는 가변인자를 제일 뒤에 위치 시켜야 한다.

```java
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

그렇다면 왜 가변인자를 사용하는 것일까? List 자료형을 활용해도 되지 않을까? 아래 코드를 살펴보자.

```java
// 가변인자 사용
public void printArgs(String... args) {

    for (String arg : args) {
        System.out.println(arg);
    }
}

// 리스트 사용
public void printArgs(List<String> args) {
    for (String arg : args) {
        System.out.println(arg);
    }
}

// -------------------------------- 함수 호출 ----------------------------------- //

// 가변인수 사용 함수 호출
printArgs("Hello", "World");

// List 사용 함수 호출
List<String> args = Arrays.asList("Hello", "World");
printArgs(args);  // List 사용
```

두 메서드는 매개변수의 타입 외에는 차이가 없고, 기능적으로 동일하다. 다만 메서드를 호출함에 있어 가변인자는 호출 코드의 가독성과 사용편의성을 높일 수 있는 장점이 있음을 확인할 수 있다.

---

https://hirodevelodiary.tistory.com/entry/Java가변인자varargs을-왜-쓰는거지-쉬운-이해와-활용

### 영속성 전이

영속성 전이는 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용한다. 영속성 전이는 연관관계를 매핑하는 것과 아무런 관련이 없다. 단지 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함만을 제공할 뿐이다. CASCADE는 두 엔티티의 생명주기기 일치하고 부모-자식 관계처럼 소유자가 단 한개 뿐일 때 사용하면 편리하다. 

![](https://velog.velcdn.com/images/yeoni_/post/6eae4fc1-8ee5-4c4a-b0a4-495fd6b34b9c/image.png)


강의 예제에서도 CASCADE를 활용하고 있다. 흐름을 생각해보자. 사용자가 주문을 하면 Order 객체가 생성이 되고 그에 파생되어 Delivery와 OrderItem 객체가 생성된다. 이처럼 Delivery 객체와 OrderItem 객체의 생명주기는 Order 객체의 생명주기에 의존적이다. 또한 위의 클래스 다이어그램에서 Delivery와 OrderItem을 살펴보면, 다른 엔티티에서 두 엔티티를 참조하지 않는 상태임을 확인할 수 있다. 이 경우에는, 아래 코드와 같이 영속성 전이(CASCADE)를 활용한다면 편리함을 얻을 수 있다.

```java
@Entity
@Table(name = "orders")
@Getter @Setter
@NoArgsConstructor
public class Order {

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
 
}
```
## 디자인 패턴

간단하게 보면, 비즈니스 로직 구현을 어디에서 하는가에 따라 아래 2가지 패턴으로 나뉘게 된다.

### 트랜잭션 스크립트 패턴

> 엔티티에 비지니스 로직이 거의 없고, **서비스 계층에서 비즈니스 로직을 처리하는 방법**

</aside>

트랜잭션 스크립트 패턴을 사용하면 엔티티는 단순히 데이터를 전달하는 역할을 하게 되지만 서비스 로직은 커지게 된다. 즉, 엔티티에는 비즈니스 로직이 거의 없으며 **서비스 계층에 비즈니스 로직이 집중**되어 있다. 일반적으로 우리에게 익숙한 방법이다. 모듈화만 잘 한다면 쉽게 개발이 가능하며 효율도 좋지만 로직이 복잡할수록 코드가 난잡해지며 코드의 중복을 막기 어렵다는 단점이 있다.

### 도메인 모델 패턴

> 대부분의 **비즈니스 로직이 엔티티 안에 구성**되어 있고, **서비스 계층은 엔티티에 필요한 역할을 위임**

엔티티 안에 비즈니스 로직을 가지고 객체지향을 활용하는 기법으로 DDD(도메인이 비즈니스 로직의 주도권을 가지고 개발하는 방법)를 접목시킬 경우 이 방법을 사용한다고 한다. 즉, 대부분의 **비즈니스 로직을 엔티티에 구현**함으로써 서비스 계층은 엔티티를 호출하는 정도의 얇은 비즈니스 로직을 가지게 된다. 객체 지향 설계를 기반으로 하기 때문에 재사용성과 확장성, 유지 보수 측면에서 장점이 있지만 도메인 모델 구축에 많은 시간과 노력이 들어가고 객체들의 관계 및 데이터베이스와의 매핑 등 고려해야 하는 점이 많다.

---

그렇다면 둘 중 무엇을 쓰는 것이 좋을까? 정답은 없다. 각자의 패턴 모두 장단점이 있기 때문에 상황에 맞게 적절한 방법을 선택하는 것이 중요하다.
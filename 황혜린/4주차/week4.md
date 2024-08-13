# 섹션 6, 7 정리

# 주문 도메인 개발

## 가변 인자(Varargs)와 주문 생성

주문 도메인에서는 가변 인자를 사용하여 주문을 생성한다. 가변 인자는 메서드의 매개변수로 여러 개의 값을 받을 수 있으며, 배열로 처리된다. 예를 들어, `Order.createOrder(member, delivery, item1, item2, item3)`처럼 여러 개의 `OrderItem` 객체를 전달할 수 있다. 이러한 방식은 코드의 가독성과 사용성을 높여준다.

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

가변 인자는 메서드 호출 시 여러 개의 인자를 간편하게 전달할 수 있는 장점이 있다. 예를 들어, `List`를 사용할 수도 있지만, 가변 인자를 사용하면 호출 시 코드가 더 간결해진다.

## 영속성 전이(CascadeType)와 주문 서비스 개발

영속성 전이(CascadeType)는 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하는 편리함을 제공한다. `Order` 엔티티의 생명주기와 연관된 `Delivery`와 `OrderItem` 엔티티는 `CascadeType.ALL`을 사용해 함께 관리한다. 이를 통해 `Order`만 저장하면 관련된 `OrderItem`과 `Delivery`도 자동으로 저장된다.

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

이처럼, `CascadeType.ALL`을 설정하면 `Order` 엔티티 하나만 persist해도 관련된 `OrderItem`과 `Delivery` 엔티티가 자동으로 persist된다. 하지만, `Delivery` 엔티티가 다른 엔티티에서도 많이 사용된다면 별도의 리포지토리를 만들어 관리하는 것이 더 안전하다.

## 디자인 패턴: 트랜잭션 스크립트 vs 도메인 모델

비즈니스 로직 구현 방법은 크게 두 가지 패턴으로 나눌 수 있다:

1. **트랜잭션 스크립트 패턴**: 엔티티에 비즈니스 로직이 거의 없고, 서비스 계층에서 모든 비즈니스 로직을 처리하는 방식이다. 이 패턴은 모듈화가 잘 되면 간단한 로직에 적합하지만, 로직이 복잡해지면 코드가 난잡해질 수 있다.
2. **도메인 모델 패턴**: 대부분의 비즈니스 로직이 엔티티 내부에 구현되어 있고, 서비스 계층은 엔티티에 필요한 역할을 위임하는 방식이다. 객체 지향 설계를 기반으로 하므로 재사용성과 유지 보수 측면에서 장점이 있지만, 도메인 모델 구축에 많은 시간과 노력이 필요하다.

## 변경 감지와 병합(Merge)

변경 감지와 병합은 모두 데이터를 수정할 때 사용하는 방식이다.

### 병합(Merge)

병합은 준영속 상태의 엔티티를 다시 영속화하는 데 사용된다. 병합을 사용하면 모든 필드가 교체되므로, 값이 없는 필드가 null로 업데이트될 위험이 있다.

```java
@Transactional
void update(Item itemParam) {
    Item mergeItem = em.merge(itemParam);
}
```

병합은 준영속 엔티티의 식별자를 기준으로 영속 상태의 엔티티를 찾아와, 모든 값을 복사한다. 그러나 병합보다는 변경 감지를 사용하는 것이 더 안전하다.

### 변경 감지(Dirty Checking)

변경 감지는 영속 상태의 엔티티를 수정할 때 가장 좋은 방법이다. 트랜잭션이 커밋될 때 JPA는 자동으로 변경 사항을 감지하여 update 쿼리를 실행한다.

```java
@Transactional
public void updateItem(Long id, String name, int price, int stockQuantity) {
    Item item = itemRepository.findOne(id);
    item.setName(name);
    item.setPrice(price);
    item.setStockQuantity(stockQuantity);
}
```

변경 감지를 사용하면 필요한 필드만 선택적으로 수정할 수 있어 효율적이며, 예상치 못한 데이터 손실을 방지할 수 있다.

## 주문 검색 기능 개발과 QueryDSL

주문 검색 기능에서는 사용자 입력에 따라 동적으로 쿼리를 생성해야 한다. 이를 위해 QueryDSL을 활용한다. QueryDSL은 빌더 패턴 스타일로 동적 쿼리를 작성할 수 있어 가독성과 객체 지향성을 높인다.

### QueryDSL 설정

QueryDSL을 사용하기 위해서는 JPAQueryFactory를 빈으로 등록해야 한다.

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

### QueryDSL을 사용한 동적 쿼리 작성

QueryDSL을 사용하면 동적 쿼리를 쉽게 작성할 수 있다. `BooleanBuilder`를 사용해 조건을 조립하며, `and`와 `or` 메서드를 통해 동적으로 쿼리 조건을 추가할 수 있다.

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final JPAQueryFactory jpaQueryFactory;

    public List<Order> findAll(OrderSearch orderSearch) {
        QOrder order = QOrder.order;
        QMember member = QMember.member;

        BooleanBuilder builder = new BooleanBuilder();

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
                .fetch();
    }
}
```

이렇게 작성된 동적 쿼리는 사용자가 입력한 검색 조건에 따라 다양한 주문 결과를 반환할 수 있다.

## 웹 계층 개발

### 뷰 템플릿 변경과 DevTools 사용

Spring Boot DevTools를 사용하면 서버를 재시작하지 않고도 뷰 템플릿 변경사항이 즉시 반영된다. `build.gradle`에 DevTools 의존성을 추가하고, `command + shift + F9`를 통해 변경사항을 바로 반영할 수 있다.

```groovy
implementation 'org.springframework.boot:spring-boot-devtools'
```

### 부트스트랩을 사용한 디자인 개선

웹 페이지의 디자인을 개선하기 위해 부트스트랩을 사용한다. 최신 버전을 [부트스트랩 공식 사이트](https://getbootstrap.com/)에서 다운로드하여 프로젝트에 추가한다. 또한, `resources/static/css/jumbotron-narrow.css`를 추가하여 더 나은 디자인을 구현할 수 있다.

### 유효성 검사와 @Valid 사용

Spring Boot에서 유효성 검사를 위해 `@Valid` 어노테이션을 사용한다. DTO 클래스 필드에 유효성 검사 어노테이션을 적용하여 입력값을 검증하며, `BindingResult` 객체를 사용해 검증 결과를 처리할 수 있다.

```java
@Getter @Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수 입니다")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}

@PostMapping("/members/new")
public String create(@Valid MemberForm form, BindingResult result) {
    if (result.hasErrors()) {
        return "members/createMemberForm";
    }
    ...
}

```

`@Valid` 어노테이션과 함께 사용된 `BindingResult` 객체는 유효성 검사 결과를 담고 있으며, 오류가 발생하면 해당 내용을 처리해준다.

## 결론

주문 도메인 개발에서는 가변 인자, 영속성 전이, 디자인 패턴, 변경 감지 및 병합의 개념을 이해하는 것이 중요하다. 또한, QueryDSL을 사용한 동적 쿼리 작성과 웹 계층 개발에서의 유효성 검사, 부트스트랩을 활용한 디자인 개선 등의 방법을 통해 안정적이고 효율적인 애플리케이션을 구현할 수 있다.

# 도메인 모델과 테이블 모델
## 도메인 분석 설계
- 회원 기능
-   회원 등록
-   회원 조회
- 상품 기능
-   상품 등록
-   상품 수정
-   상품 조회
- 주문 기능
-   상품 주문
-   주문 내역 조회
-   주문 취소
- 추가 기능
-   재고 관리
-   도서, 음반, 영화 관리
-   배송 정보 관리
## 테이블 설계
<img width="764" alt="image" src="https://github.com/user-attachments/assets/9e57c803-7e90-48a8-8b00-2ca77148f7c5">

### 양방향 연관관계 관련 주의 사항
가능하면 양방향 연관관계를 사용하지 않는 것이 좋지만, 다양한 연관관계를 표현하기 위해 필요할 때 사용한다. 예를 들어, Member와 Order 사이의 1:N 관계는 필요하지 않다. 주문이 회원을 참조하는 것으로 충분하다.

### 임베디드 타입
JPA는 데이터 타입을 엔티티 타입과 값 타입으로 나눈다. 엔티티 타입은 @Entity 어노테이션을 통해 정의하며, 식별자를 통해 추적이 가능하다. 값 타입은 int, Integer, String처럼 단순히 값으로 사용하는 기본 타입이다. 값 타입 중 임베디드 타입은 여러 값을 모아서 하나의 객체로 사용하는 복합 값 타입이다. 임베디드 타입은 연관된 속성을 묶어서 관리하여 응집도를 높이고 재사용성을 높일 수 있다. 예를 들어, 주소 정보를 Address 객체로, 근무시간을 Period 객체로 만들어 사용한다.

### 상속 관계 테이블 매핑
객체의 상속 구조를 관계형 데이터베이스의 슈퍼타입 서브타입 관계로 매핑해야 한다. 단일 테이블 전략은 상속 구조의 모든 데이터를 하나의 테이블에서 관리하되, DTYPE 속성을 두어 객체들을 구분하는 방식이다. 이는 조인 쿼리가 나가지 않아 성능상의 이점이 있다.

### 연관관계의 주인
연관관계의 주인은 양방향 매핑에서 외래 키를 관리하는 객체를 말한다. 주인이 아닌 객체는 읽기만 가능하다. 외래 키가 있는 객체를 연관관계의 주인으로 설정해야 하며, 일대다 관계에서는 항상 다쪽에 외래키가 위치하도록 설계해야 한다.

## 엔티티 클래스 개발
### 임베디드 타입 구현
Address 클래스 위에 @Embeddable 어노테이션을 붙이고, 이를 사용하는 Member 클래스의 필드에는 @Embedded 어노테이션을 붙인다. 엔티티나 임베디드 타입의 경우 자바의 기본 생성자를 protected로 설정하는 것이 좋다.

```
@Embeddable
public class Address {
    private String city;
    private String street;
    private String zipcode;

    protected Address() {}
    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}

@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;
    private String name;
    @Embedded
    private Address address;
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```

### 연관관계 주인 설정
양방향 연관관계에서 연관관계의 주인이 아닌 객체는 mappedBy를 사용해 매핑하는 주체가 아님을 나타낸다. 연관관계의 주인은 외래키가 있는 객체로 설정해야 하며, 일대일 매핑의 경우 더 많이 접근하는 객체를 연관관계의 주인으로 설정한다.

```
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "order_id")
    private Long id;
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();
    @OneToOne
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
    private LocalDateTime orderDate;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```

### 열거형 매핑
@Enumerated 어노테이션을 통해 열거형을 데이터베이스에 매핑한다. EnumType.STRING 방식을 사용하여 Enum의 이름을 DB에 저장한다.

```
@Enumerated(EnumType.STRING)
private DeliveryStatus status;
```

### 계층형 구조
부모 자식 간의 관계를 깊이로 구분하여 표현하는 구조이다. 카테고리는 여러 자식을 가질 수 있으며, 자식은 한 명의 부모를 가진다.

```
@Entity
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "category_id")
    private Long id;
    private String name;
    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;
    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();
}
```

### 엔티티 설계 시 주의점
**엔티티에는 가급적 Setter를 사용하지 말자**
값 타입은 불변하게 설계해야 하며, 생성할 때만 값이 세팅되도록 해야 한다.

**모든 연관관계는 지연로딩으로 설정**
모든 연관관계를 지연로딩(LAZY)으로 설정해야 한다. @ManyToOne과 @OneToOne은 기본 설정이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.
```
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "member_id")
private Member member;
```

**컬렉션은 필드에서 초기화하자**
컬렉션은 필드에서 바로 초기화하여 NullPointerException을 방지하고, Hibernate가 컬렉션을 읽지 못하는 문제를 방지할 수 있다.
```
private List<OrderItem> orderItems = new ArrayList<>();
```

**연관관계 편의 메서드 작성**
연관관계 편의 메서드는 양방향 관계를 한 번에 설정하는 메서드이다.
```
public void setMember(Member member) {
    this.member = member;
    member.getOrders().add(this);
}
```

**다대다 관계를 풀어내기**
다대다 관계를 중간 테이블(ORDERITEM)을 사용하여 풀어낸다. ORDER와 ITEM 사이의 관계를 ORDERITEM 테이블로 풀어낸다.

**일대다 양방향 관계**
연관 관계의 주인의 반대쪽 필드는 mappedBy를 설정하여 읽기 전용으로 사용한다.

**상속관계**
Item 엔티티를 Movie, Album, Book 등으로 상속받아 사용한다.
```
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
```

**1:1 관계**
Order와 Delivery는 1:1 관계로 Order 엔티티에서 delivery FK를 갖는 것이 바람직하다.
```
@OneToOne(cascade = CascadeType.ALL,fetch = FetchType.LAZY)
@JoinColumn(name = "delivery_id")
private Delivery delivery;
```
```
@OneToOne(mappedBy = "delivery",fetch = FetchType.LAZY)
private Order order;
```

**N:M 관계**
실무에서는 다대다 관계를 사용하지 않고, 중간에 category_item 테이블을 둔다.
```
@ManyToMany
@JoinTable(name = "category_item",
           joinColumns = @JoinColumn(name = "category_id"),
           inverseJoinColumns = @JoinColumn(name = "item_id"))
private List<Item> items = new ArrayList<>();
```
```
@ManyToMany(mappedBy = "items")
private List<Category> categories = new ArrayList<>();
```

**CascadeType**
cascade = CascadeType.ALL은 상위 엔터티에서 하위 엔터티로 모든 작업을 전파한다.
```
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
private List<OrderItem> orderItems = new ArrayList<>();
```

### JPA의 엔티티는 왜 기본 생성자가 있어야 할까?
JPA는 DB 값을 객체 필드에 주입할 때 기본 생성자로 객체를 생성한 후 Reflection API를 사용하여 값을 매핑한다. 기본 생성자가 없다면 Reflection은 객체를 생성할 수 없기 때문에 기본 생성자가 필요하다.

## 도메인 모델과 테이블 모델
![](https://velog.velcdn.com/images/yeoni_/post/f9feb2bd-6695-4cea-a4b5-7c3fc7e1c01c/image.png)

### 임베디드 타입

최상위 레벨로 보면 JPA는 데이터 타입을 두 가지로 분류한다. 바로 **엔티티 타입**과 **값 타입**이다. 엔티티 타입은 우리가 구현할 때 `@Entity` 어노테이션을 붙여 정의하는 객체이다. 엔티티 내부의 모든 값들을 바꿔도 식별자만 유지되면 추적이 가능하다. 반면 값 타입은 int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체를 말한다. 식별자가 없고 값만 있으므로 변경시 추적 불가능하다. 임베디드 타입은 값 타입 중 하나이다. 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 한다. 

![](https://velog.velcdn.com/images/yeoni_/post/cc71aacc-240d-4e42-a0f3-7ca4697693ee/image.png)


위와 같은 테이블이 있다고 생각해보자. 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가지고 있다. 그런데 테이블을 살펴보니 연관시켜서 한번에 관리하는 것이 더 수월하다는 생각이 든다. 즉,

- city, street, zipcode는 **주소**로 합칠 수 있을 것 같다.
- 근무 시작일, 근무 종료일은 **근무시간**으로 합칠 수 있을 것 같다.

![](https://velog.velcdn.com/images/yeoni_/post/bcd40fc8-00c4-4f38-8693-89d13a8103c3/image.png)

이런 식으로 회원 엔티티를 좀 더 추상화시킬 수 있다.

여기서 사용하는 Period와 Address가 바로 임베디드 타입이다. 임베디드 타입을 통해 연관된 속성들을 한번에 관리할 수 있어 응집도를 높일 수 있으며 다른 객체에서도 사용 할 수 있어 재사용성을 높힐 수 있다. 

위와 같이 임베디드 타입을 통해 객체를 분리하더라도 테이블은 하나만 매핑된다. 즉, 임베디드 타입을 사용하든 안하든 DB 테이블 입장에서는 변경되는 것이 없다는 말이다.


### 상속 관계 테이블 매핑 방법

![](https://velog.velcdn.com/images/yeoni_/post/aa639ecd-2c16-4390-8c1e-2ce13f708836/image.png)
해당 강의의 예제에는 옆과 같이 상속 관계의 객체들이 존재한다. 하지만 관계형 데이터베이스에는 상속 관계가 존재하지 않는다. 때문에 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 상속관계 매핑 작업이 필요하다.

    슈퍼타입과 서브타입이란? 

    슈퍼타입 : 상호 배타적인 더 작은 그룹으로 분할 시킬 수 있는 엔티티
    서브타입 : 슈퍼타입 내의 분해된 그룹
     

총 3가지의 방법이 있지만, 강의에서 사용하는 방식인 **단일 테이블 전략**에 대해서 알아보자.

![](https://velog.velcdn.com/images/yeoni_/post/4567d00b-ccf6-472c-896f-68f1ca343a3c/image.png)

**단일 테이블 전략**이란, 각각의 테이블로 나누는 것이 아닌, 하나의 통합된 테이블로 관리하는 전략이다. 

즉, 하나의 테이블로 관리하되 DTYPE 속성을 두어 구분하는 것이다. 

단일 테이블 전략은 모든 데이터를 하나의 테이블에서 관리하기 때문에 각각의 (자식)객체들을 구분할 수 있는 방법이 없다. 때문에 반드시 DTYPE을 두어 객체들을 분류할 수 있도록 해야한다.

단일 테이블 전략은 조인 쿼리가 나가지 않기 때문에 성능상의 이점이 있다. 때문에 서비스 규모가 크지 않고, 굳이 조인 전략을 선택해서 복잡하게 갈 필요가 없다고 판단 될 때에는 단일 테이블 전략도 하나의 선택사항이 될 수 있겠다.



### 연관관계의 주인

<aside>
⚠️ **외래 키가 있는 곳을 연관관계의 주인으로 정해라**

</aside>

**연관관계의 주인**이란, **양방향 매핑에서** 두 객체 중 **외래 키**를 누가 **관리하는 객체**를 말한다. 주인이 아닌 객체는 읽기만 가능하다. 즉, 연관관계의 주인은 단순히 외래 키를 누가 관리하느냐의 문제이기 때문에 비즈니스상 우위에 있다고 주인으로 설정해서는 안된다. 이때, 일대다 관계에서 **외래키는 항상 다쪽에 위치하도록 설계**해야 한다.

![](https://velog.velcdn.com/images/yeoni_/post/2039389b-dd86-4b6d-acf1-7b8ee4a72bcf/image.png)

만약, 위와 같은 연관관계가 존재한다고 가정해보자. 이때, Member(**多**)가 아닌 Team이 연관관계의 주인이 된다면 어떨까? 해당 팀에 소속된 member에 변경이 생기게 된다면 본인의 테이블인 Team이 아닌 다른 테이블 즉, Member 테이블에 Update 쿼리가 나가게 된다. 즉, Team 객체에 행위를 하였는데 다른 객체인 Member의 상태가 변하는 것이다. 이외에도 성능 문제 등, **연관관계의 주인은 외래키가 있는 객체**가 되어야 한다.


## 엔티티 클래스 개발
### 임베디드 타입 구현

아래 코드에서 볼 수 있듯이 임베디드 타입임을 정의하는 Address 클래스 위에는  `@Embaddable` 어노테이션을 붙인다. 그리고 이 임베디드 타입을 사용하는 Member 클래스의  필드에는 `@embadded` 어노테이션을 붙인다. 둘 중 하나만 사용해도 정상적으로 작동하지만 모든 클래스에서 해당 타입이 내장 타입인 것을 가시적으로 확인하기 위해 두개의 어노테이션 모두 쓰는 것을 권장한다.

```java
// Member 클래스
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    Address address;

    @OneToMany(mappedBy = "member")
    List<Order> orders = new ArrayList<>();
}

// Address 클래스
public class Address {

    private String city;

    private String street;

    private String zipcode;

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}
```

더하여 JPA 스펙상 엔티티나 임베디드 타입의 경우 자바의 기본 생성자를 `Protected`로 설정하는 것이 좋다.
`@NoArgsConstructor(access = AccessLevel.*PROTECTED*)` 을 통해 해당 엔티티의 접근 권한을 `Protected`로 설정할 수 있다.

---

https://velog.io/@ohzzi/JPA의-엔티티에-protected-public-기본-생성자가-필요한-이유

### 연관관계 주인 설정 방법

```java
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

앞에서 연관관계 주인의 개념에 대해서 배워보았다. 그렇다면 해당 객체가 연관관계의 주인임을 코드에서 어떻게 표현할 수 있을까? 양방향 연관관계에서 연관관계의 주인이 아니라면 `mappedBy`를 통해 해당 객체는 매핑하는 주체가 아님을 드러낼 수 있다. 즉, Member 객체와 Order 객체에서 연관관계의 주인은 Order 객체이기 때문에 Member 객체에서 orders 필드에 `@OneToMany(mappedBy = "member")` 를 추가하여, Order 객체의 member 필드에 의해 매핑이 되었음을 반드시 나타내주어야 한다. 이를 통해 연관관계의 주인이 아닌 객체에서는 해당 데이터를 읽을 수만 있고 직접적인 변경이나 추가는 불가능하다.
추가적으로 일대일 매핑의 경우 연관관계의 주인을 어느 객체로 설정할지 모호할 수 있다. 해당 예시에서는 Delivery 객체와 Order 객체가 일대일 관계인데, 그렇다면 어느 객체를 연관관계의 주인으로 설정해야 할까? 
이런 경우에는 **더 많이 접근하는 객체를 연관관계의 주인으로 설정**하는 것이 좋다. 데이터를 불러올 때 , Delivery에 직접 접근하는 경우보다는 Order에 접근하는 경우가 더 많기 때문에 Order 객체를 연관관계의 주인으로 설정하는 것이다.

### 열거형 매핑 방법

```java
@Enumerated(EnumType.STRING)
private DeliveryStatus status;
```

`@Enumerated` 어노테이션은 JPA에서 열거형(enum) 타입을 데이터베이스에 매핑할 때 사용하는 어노테이션이다. `@Enumerated` 어노테이션은 두 가지 매핑 전략을 지원한다. 바로 `EnumType.ORDINAL`과 `EnumType.STRING`이다. 하지만 `ORDINAL`방식은 사용을 하지 않는 것이 좋다. `Ordinal` 방식은 해당 객체가 선언된 순서에 따라 증가하며 데이터베이스에 정수 값으로 저장되는 방식이다. 하지만 이런 순서 의존적인 방식은 데이터의 무결성을 해치고 예상치 못한 버그를 발생시킬 수 있다. 때문에 `EnumType.String` 방식을 이용하여 Enum에 선언된 상수의 이름을 String 클래스 타입으로 변환하여 DB에 저장하는 방식을 사용해야 한다.

### 계층형 구조

**계층형 구조**는 **부모 자식 간의 관계를 깊이로 구분**하여 표현하는 구조이다.

```java
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

관계를 살펴보면, 하나의 카테고리는 여러 자식을 가질 수 있으며 하나의 자식은 한 명의 부모를 가질 수 있다. 즉, Category parent가 한명이고 List<Category> child는 여러명이므로 `@ManyToOne` 과 `@OneToMany`로 관계를 잡아주어야 한다.

---

https://kkalgo.tistory.com/171

## 엔티티 설계시 주의점

### 엔티티에는 가급적 Setter를 사용하지 말자.

```java
public class Address {

    private String city;

    private String street;

    private String zipcode;

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}
```

Address와 같은 값 타입은 기본적으로 불변하게 설계되어야하기 때문에 생성할 때만 값이 세팅이 되게 하여야 한다. 즉 setter를 사용하지 않고 변경이 불가능하도록 만들어야한다. 변경이 필요할 경우에는 Setter를 사용하기보다는 변경 지점이 명확하도록 변경을 위한 비즈니스 메소드를 별도로 작성하는 것이 좋다.

---

https://velog.io/@backfox/setter-쓰지-말라고만-하고-가버리면-어떡해요

### 모든 연관관계는 지연로딩으로 설정

실무에서는 모든 연관관계를 지연로딩(*LAZY*)로 설정해야 한다. `@OneToMany` 랑 `@ManyToMany` 는 기본 설정이 지연로딩이지만 `@ManyToOne` 랑 `@OneToOne` 는 기본 설정이 즉시로딩이기 때문에 직접 지연로딩으로 아래와 같이 설정해야 한다. → 자세한 내용은 [아티클](https://www.notion.so/JPA-N-1-1a8ae4c21cd74f05be1d03861707f00b?pvs=21) 참고

```java
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```

### 컬렉션은 필드에서 초기화하자

```java
private List<OrderItem> orderItems = new ArrayList<>();
```

컬렉션은 필드에서 바로 초기화하는 것이 안전하다. 2가지 이유가 있는데 먼저 **`NullPointerException`을 방지**할 수 있다. 만약 필드에서 초기화를 하지 않았다고 가정해보자. 누군가가 기본 생성자를 통해 해당 필드 값을 초기화하지 않고 객체를 생성한다면 필드 값의 상태는 *NULL*이 될 것이다. 그리고 이런 *NULL* 상태의 컬렉션 필드를`addAll()` 로 할당하면 무슨 문제가 발생할까? `NullPointerException` 이 터진다.

![](https://velog.velcdn.com/images/yeoni_/post/ecd9a37f-8c08-4751-9af4-aa274bcacd69/image.png)

위의 사진에서 확인할 수 있듯 `addAll()` 은 NULL 값이 허용되지 않는다. 때문에 이런 오류를 방지하기 위해서 컬렉션은 필드에서 초기화하는 것이 좋다.
두번째 이유로는 필드에서 초기화를 하지 않는다면 **Hibernate가 컬렉션을 읽지 못할 수 있다.** Hibernate가 엔티티를 영속화 할 때 내부에서 컬렉션이 있으면 Hibernate가 특별하게 조작한 자신만의 내장 컬렉션으로 변경한다. 그런데 개발자가 임의로 나중에 `new ArrayList`로 초기화를 하게 되면 이 부분이 Hibernate가 관리하는 컬렉션에서 개발자가 직접 만든 컬렉션으로 변경될 수 있고 정상 동작하지 않을 수 있다. 이런 문제를 방지하기 위해 필드에서 빠르게 컬렉션을 초기화 하고, 해당 컬렉션을 바꾸는 행위를 막도록 코드를 작성하는 것이 좋다.

### 연관관계 편의 메서드 작성

연관관계 편의 메서드는 한 번에 양방향 관계를 설정하는 메서드이다. 객체의 양방향 연관관계는 양쪽 모두 관계를 맺어주어야 한다. 이때, 연관관계 메서드를 따로 설정해주지 않으면 어떻게 될까?

```java
Member member = new Member();
Order order = new Order();

member.getOrders.add(order);
order.setMember(member);
```

위의 코드처럼 하나하나 수작업으로 설정해주어야 한다. 번거러울 뿐더러 실수로 한 줄을 넣지 않게 되면 둘 중 하나만 호출이 되어 양방향이 깨질 수 있다. 때문에 아래와 같이 연관 관계 편의 메서드를 작성하여 하나인 것 처럼 사용하는 것이 안전하다.

```java
// 연관관계 편의 메서드들
public void setMember(Member member) {
    this.member = member;
    member.getOrders().add(this);
}

public void addOrderItem(OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder(this);
}

public void setDelivery(Delivery delivery) {
    this.delivery = delivery;
    delivery.setOrder(this);
}
```
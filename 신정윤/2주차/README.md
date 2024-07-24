## 도메인 분석 설계

- 회원기능
    - 회원등록
    - 회원조회
- 상품기능
    - 상품등록
    - 상품수정
    - 상품조회
- 주문기능
    - 상품주문
    - 주문내역조회
    - 주문 취소
- 추가
    - 재고 관리
    - 도서, 음반, 영화
    - 배송정보


### 테이블 설계

- 가능하면 양방향 연관관계는 쓰지않는게 좋다. → 그러나 다양한 연관관계를 표현하기 위해 사용함
- 예를 들어, Member와 Order는 사이에 1:N 관계는 굳이 필요없다. 주문이 회원을 참조하는 것으로 충분함.
- **참고: 외래 키가 있는 곳을 연관관계의 주인으로 정해라.**
    - 일대다 관계라면 다쪽에 외래키가 존재하므로, 연관관계의 주인이 된다.

### 🤔 다대다 관계를 풀어내기

- Order - OrderItem - Item
    
    [주문과 상품(물품)의 관계 - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/653311/주문과-상품-물품-의-관계)
    
    - 다대다를 사용하지 않고 중간 테이블(ORDERITEM)을 사용하여 설계하신것으로 이해하시면 됩니다.
        
        ORDER -(1:N) - ORDERITEM - (N : 1) - ITEM
        
    - 주문시 여러 상품 선택가능, 같은 상품도 여러 번 주문될 수 있음
- Category - Category Item -Item
    - 다대다를 일대다, 다대일로 풀어냄

### 🤔 일대다 양방향 관계

- FK를 가진 엔티티는 실제 값을 변경하거나 하는 용도로 사용
- 연관 관계의 주인의 반대쪽 필드는 mappedby를 설정하여 읽기 전용으로 쓰게 된다.

### 엔티티 클래스 개발

- Order 엔티티
    - name = “member_id”는 FK이름을 의미
        
        ```java
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "member_id")
        ```
        

- Address(임베디드 타입)
    - `@Embedded` `@Embeddable`  둘 중에 하나만 써도 되지만, 김영한님은 그냥 둘 다 쓰신다고 하심.
    - 기본생성자란 기본 생성자는 어떠한 매개변수도 전달받지 않으며, 기본적으로 아무런 동작도 하지 않음.
        
    ![image](https://github.com/user-attachments/assets/052283db-93a2-48f0-add8-22d746490565)
        
        ```
        @Embeddable
        @Getter
        public class Address {
            private String city;
            private String street;
            private String zipcode;
        
            protected Address() {
            }
        
            public Address(String city, String street, String zipcode) {
                this.city = city;
                this.street = street;
                this.zipcode = zipcode;
            }
        
        }
        ```
        

- 상속관계
    - Item 엔티티(Movie, Album, Book)
    
    ```java
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn(name = "dtype")
    ```
    

### 🤔 1:1 관계

- Order -  Delivery
    - 주문에서 배송을 확인하는 경우가 배송에서 주문을 확인하는 경우가 많다.
    - 따라서 Order엔티티에서 delivery FK를 갖는 것이 바람직함.
    
    ```java
    @OneToOne(cascade = CascadeType.ALL,fetch = FetchType.LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
    ```
    
    ```java
    @OneToOne(mappedBy = "delivery",fetch = FetchType.LAZY)
    private Order order;
    
    ```
    

### 🤔M : N 관계

- ❗️실무에서 사용 안 함❗️
- Item - Category
- 실제로 다대다 관계가 불가능하기 때문에, 중간에 category_item이라는 테이블을 둔다.
- joinColumns과 inverseJoinColumns
    - 

```java
@ManyToMany
@JoinTable(name = "category_item",joinColumns = @JoinColumn(name = "category_id"),
inverseJoinColumns = @JoinColumn(name = "item_id"))
private List<Item> items = new ArrayList<>();
```

```java

@ManyToMany(mappedBy = "items")
private List<Category> categories = new ArrayList<>();

```

### 모든 연관관계는 지연로딩으로 설정!!

- 즉시로딩( EAGER )은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행할 때 N+1
문제가 자주 발생한다.
- @XToOne(OneToOne, ManyToOne) 관계는 기본이 즉시로딩이므로 직접 지연로딩으로 설정해야 한다.

### CascadeType

`cascade = CascadeType.*ALL` 은*  상위 엔터티에서 하위 엔터티로 모든 작업을 전파한다는 의미이다.

따라서 아래코드에서 order가 저장될 때 orderItems도 한꺼번에 저장된다.(persist(order)만 수행)

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();
```

### **연관관계 편의 메소드**

## 🧐 더 알아보기

### **JPA의 엔티티는 왜 기본 생성자가 있어야 할까요?**

[JPA의 엔티티에 protected, public 기본 생성자가 필요한 이유](https://velog.io/@ohzzi/JPA의-엔티티에-protected-public-기본-생성자가-필요한-이유)

Spring Data JPA 에서 Entity에 기본 생성자가 필요한 이유는 동적으로 객체 생성 시 `Reflection API`
를 활용하기 때문이다.

JPA는 DB 값을 객체 필드에 주입할 때 기본 생성자로 객체를 생성한 후 `Reflection API`를 사용하여 값을 매핑한다.

때문에 기본 생성자가 없다면 **`Reflection`은 해당 객체를 생성 할 수 없기 때문에** JPA의 Entity에는 기본 생성자가 필요하다.

- 그렇다면 왜 모든 엔티티에 기본생성자를 만드지 않은걸까?
    
    [기본 생성자에 관해 질문드립니다. - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/105043/기본-생성자에-관해-질문드립니다)

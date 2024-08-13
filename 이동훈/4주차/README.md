# **주문 도메인 개발**

## 비즈니스 로직의 위치에 따라…

### 도메인 모델 패턴

**정의**

- **비즈니스 로직이 엔티티 안에 구성**되어 있고, **서비스 계층은 엔티티에 필요한 역할을 위임하는 것으로** 보통 DDD를 접목시킬 경우 이 방법을 사용한다.

**장점**

- 구현이 매우 쉽다.
    - 그 이유는 트랜잭션 스크립트 방식의 구현 방법의 단순함 때문이다.
    - MVC 패턴과 커맨드 패턴을 함께 사용하더라도 이러한 단순함은 그대로 유지되기 때문에, 스트러츠 코드에 조금만 익숙해지면 손쉽게 스트러츠로도 트랜잭션 스크립트 패턴을 구현할 수 있게 된다.

**단점**

- 트랜잭션 스크립트로 구성된 어플리케이션은 비즈니스 로직이 복잡해질수록 난잡한 코드를 만들게 된다.
    - 애초에 도메인에 대한 분석/설계 개념이 약하기 때문에 코드의 중복 발생을 막기 어려워진다.
    - 또한, 쉬운 개발에 익숙해지기 때문에 공통된 코드를 공통 모듈로 분리하지 않고 복사&붙이기 방식으로 중복된 코드를 만드는 유혹에 빠지기 쉽다.


### 트랜잭션 스크립트 패턴

**정의**

- 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것

**장점**

- 객체 지향에 기반한 재사용성, 확장성, 그리고 유지 보수의 편리함에 있다.
    - 일단 도메인 모델을 구축하고나면 약간의 수정은 필요하겠지만 언제든지 재사용할 수 있다.
    - 또한, 인터페이스에서 더 나아가 컴포넌트 개념을 바탕으로 도메인 모델을 개발하게 되면 무한한 확장성을 갖게 된다.

**단점**

- 하나의 도메인 모델을 구축하는 데 많은 노력이 필요하다.
    - 객체를 판별해내야 하고 객체들 간의 관계를 정리하고 설계하며, 객체와 데이터베이스 사이의 매핑이 필요하기 때문이다.
    - 이는 이론과 경험을 함께 겸비하지 않으면 쉽게 풀수 없는 문제이기 때문에, 도메인 모델에 능숙한 개발자가 팀에 없을 경우 도메인 모델을 구축하는 것 자체가 힘들어질 수도 있다.

## **가변 인자(Varargs)**

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

- 가변인자는 메서드의 매개변수를 동적으로 처리할 수 있도록 해준다.
- 위의 createOrder() 메서드에 'OrderItem...'은 가변인자를 나타내는 문법이다.
- 따라서, OrderItem이라는 객체 배열이 선언되고 메서드가 호출될 때 넘겨받은 인자들이 이 배열로 묶어 처리된다.

### 가변인자와 리스트 비교

OrderItem 객체을 여러 개 매개변수로 넣으려면 List형태를 활용하면 되지 않을까? 라는 의문이 들 수 있다.

List를 사용해서도 가능하긴 하다. 다만 가변인자('...')를 사용하면 인자의 개수가 가변적일때 간단하게 처리할 수 있다는 장점이 있다.

```java
public void printArgs(String... args) {

    for (String arg : args) {
        System.out.println(arg);
    }
}

public void printArgs(List<String> args) {
    for (String arg : args) {
        System.out.println(arg);
    }
}
```

- 두 메서드는 매개변수의 타입외에는 차이가 없고, 기능적으로 동일하다.

```java
// 가변인수 사용
printArgs("Hello", "World");

// List사용
List<String> args = Arrays.asList("Hello", "World");
printArgs(args);  // List 사용
```

- 다만 해당 메서드를 호출함에 있어서 가변인자는('...')는 호출 코드의 가독성과 사용편의성을 높일 수 있는 장점이 있다.

## QueryDSL

- QueryDSL은 하이버네이트 쿼리 언어(HQL: Hibernate Query Language)의 쿼리를 타입에 안전하게 생성 및 관리해주는 프레임워크이다.

### JPQL vs QueryDSL

```java
// JPQL
String jpql = "select m from Member m where m.username = :username";

List<Member> result = em.createQuery(query, Member.class).getResultList();

// QueryDSL
String username = "java";

List<Member> result = queryFactory
        .select(member)
        .from(member)
        .where(usernameEq(username))
        .fetch();
```

### 장점

- 문자가 아닌 코드로 쿼리를 작성할 수 있어 컴파일 시점에 문법 오류를 확인할 수 있다.
- 인텔리제이와 같은 IDE의 자동 완성 기능의 도움을 받을 수 있다.
- 복잡한 쿼리나 동적 쿼리 작성이 편리하다.
- 쿼리 작성 시 제약 조건 등을 메서드 추출을 통해 재사용할 수 있다.
- JPQL 문법과 유사한 형태로 작성할 수 있어 쉽게 적응할 수 있다
# 섹션2

## 도메인 모델과 테이블 설계

### 양방향 연관관계에서의 **연관관계의 주인?**

**→ 외래(FK)가 있는 곳. 아래 그림에서는 ‘Orders’**

![image](https://github.com/user-attachments/assets/2ce806cf-b642-4b20-a733-999aa6c729df)


Orders에 값을 세팅해야 값이 변경되고, 거울이 되는 Member는 조회하는 용도로만 쓴다.

## 엔티티 클래스 개발1

### `@Embedded` , `@Embeddable`

> JPA(Java Persistence API)에서 값 타입(Value Type)과 관련된 기능을 지원
> 
- 값 타입(Value Type) 이란?
    - 엔티티의 일부로, 엔티티에 포함되지만 독립적으로 식별되지 않는 객체
    - 식별자(ID)가 없고, 엔티티가 아닌 객체로 취급
    - 주소(Address), 이름(Name), 기간(Period) 등등
- `@EmBeddable`
    - 값 타입 클래스를 정의할 때 사용한다.
- `@Embedded`
    - 엔티티 클래스에서 값 타입을 포함할 때 사용한다.

---

### `@Inheritance` , `@DiscriminatorValue` , `@DiscriminatorColumn`

> 상속관계 매핑을 위해 사용하는 어노테이션
> 
- 사용 예시 : `@Inheritance(strategy = InheritanceType.***SINGLE_TABLE***)`

| InheritanceType | 설명 |
| --- | --- |
| SINGLE_TABLE | - 한 테이블에 다 저장하고, DTYPE으로 구분
- 서비스 규모가 크지 않고, 굳이 조인 전략을 선택해서 복잡하게 갈 필요가 없다고 판단 될 때
- 한 테이블에 모든 컬럼을 저장하기 때문에, DTYPE 없이는 테이블 판단 불가 |
| JOINED | - 슈퍼타입, 서브타입 논리모델을 각각 테이블로 옮긴 방식 
- 테이블이 구분되어 있기 때문에 데이터를 조회할 때 조인이 필요 |
| TABLE_PER_CLASS | - 슈퍼타입은 테이블로 매핑하지 않고 서브타입만 테이블로 매핑
- SINGLE_TABLE 전략에서는 불가능한 not null 제약조건 사용 가능
- 실무에서 사용할 수 없다! |

![image](https://github.com/user-attachments/assets/1eb9cadc-de15-4e17-9608-26dd0e1287eb)


- `@DiscriminatorColumn`
    - default 값으로 "DTYPE". 이를 통해 어떤 서브타입의 데이터인지 구분
    - 부모 클래스에 선언.
- `@DiscriminatorValue`
    - default 값으로 클래스 이름.
    - 하위 클래스에 선언.

    

---

### `@Enumerated`

EnumType에는 enum 이름 값을 DB에 저장하는 `EnumType.STRING` 과 

enum 순서(숫자)값을 DB에 저장하는 `EnumType.ORDINAL` 이 있지만, 

안정성을 위해 **`@Enumerated(EnumType.STRING)` 을 쓰도록 하자!**

---

### `@OneToOne` 일대일 관계에서 FK는 어디에다?

→ 주로 access해서 *직접 조회를 많이 하는* 쪽에 둔다. 

(예제에서는 Delivery로 Order를 조회하는 일은 거의 없으므로, Order쪽에 FK를 두었다.)

## 엔티티 클래스 개발2

### `@ManyToMany` 다대다

일대 다, 다대 다 로 풀어내는 중간 테이블이 필요하다. → CATEGORY_ITEM

![image](https://github.com/user-attachments/assets/3d53e8f8-981d-45dd-8ed0-a324d51e87bf)

But, 실무에서는 잘 안쓴다! 된다 정도만 알고 넘어가자.

## 엔티티 설계시 주의점

### Setter 사용은 지양하자!

변경 포인트가 많아서 유지보수가 어려움..

### 모든 연관관계는 지연로딩으로 설정하자!

- **`@**XTo**One`** 은 디폴트가 `EAGER` 이기 때문에, 모두 **`fetch = FetchType.LAZY`**로 직접 설정해 주어야 한다.
    - EAGER 은 연관된 엔티티를 즉시 로드하므로, 불필요한 데이터베이스 조회로 성능 저하를 유발할 수 있다. 따라서 연관관계를 지연 로딩(`FetchType.LAZY`)으로 설정하면, 실제로 해당 연관 엔티티가 필요할 때만 데이터베이스에서 조회한다.
- `@XToMany` 는 디폴트가 LAZY이므로 상관 없다.

### 컬렉션은 필드에서 바로 초기화 하자!

null 참조로 인해 `NullPointerException`이 발생할 위험성을 막을 수 있다.

```java
@Entity
@Getter @Setter
public class Category {
 '''
    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();
 '''
}
```

### `CascadeType`

| CascadeType | 설명 |
| --- | --- |
| ALL | 모든 Cascade를 적용한다. |
| PERSIST | 엔티티를 영속화할 때, 연관된 하위 엔티티도 함께 유지한다. |
| MERGE | 엔티티 상태를 병합(Merge)할 때, 연관된 하위 엔티티도 모두 병합한다. |
| REMOVE | 엔티티를 제거할 때, 연관된 하위 엔티티도 모두 제거한다. |
| DETACH | 영속성 컨텍스트에서 엔티티를 제거한다. 부모 엔티티를 detach() 수행하면, 연관 하위 엔티티도 detach()상태가 되어 변경 사항을 반영하지 않는다. |
| REFRESH | 상위 엔티티를 새로고침(Refresh)할 때, 연관된 하위 엔티티도 모두 새로고침한다. |

### 연관관계 메서드

양방향 관계를 설정하는 메서드.  (기본편에서 자세하게설명한다)

```java
Member member = new Member();
Order order = new Order();

member.getOrders.add(order);
order.setMember(member);
```

위의 코드에서 실수로 두 줄 중 하나를 빼먹으면 하나만 호출이 되어 양방향이 깨질 수 있으므로, 아래와 같이 연관관계 메서드를 작성하여 사용하는 것이 좋다.

```java
public void setMember(Member member) {
    this.member = member;
    member.getOrders().add(this);
}
```

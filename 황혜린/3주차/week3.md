# 섹션 3, 4, 5 정리

## 계층형 아키텍처

### 아키텍처 구성

Spring Boot 애플리케이션은 아래와 같은 계층형 구조를 가집니다:

- **Controller** : 웹 계층으로, 클라이언트의 요청을 처리하고 응답을 반환합니다.
- **Service** : 비즈니스 로직과 트랜잭션을 처리하는 계층입니다.
- **Repository** : JPA를 직접 사용하는 계층으로, 엔티티 매니저(Entity Manager)를 통해 데이터베이스에 접근합니다.
- **Domain** : 엔티티 클래스가 모여 있는 계층으로, 모든 계층에서 사용됩니다.

## 회원 서비스 개발

### 회원 서비스 코드

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }
}
```
위 코드는 회원 가입 기능을 제공하는 서비스 계층입니다.

`@Transactional(readOnly = true)` : 데이터의 변경이 없는 읽기 전용 메서드에 사용하는 어노테이션입니다. 영속성 컨텍스트를 flush하지 않으므로 성능적인 면에서 약간의 이점이 있습니다.

### 의존성 주입 방식
Spring에서는 여러 방식으로 의존성 주입을 할 수 있습니다. 그 중 필드 주입과 생성자 주입 방식에 대해 알아보겠습니다.

**필드 주입**
```
@Service
@Transactional(readOnly = true)
public class MemberService {
	@Autowired
    private MemberRepository memberRepository;
}
```
필드 주입(Field Injection)은 멤버 객체에 @Autowired를 붙여 주입받는 방법입니다. 그러나 이 방식에는 여러 단점이 있습니다:
- final 키워드를 사용할 수 없어, 중간에 MemberRepository 객체가 변경될 가능성이 있습니다.
- 테스트 코드 작성 시, 스프링의 도움 없이 객체를 생성하기 어려워 프로젝트 전체를 실행해야 합니다.

**생성자 주입**
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
}
```
생성자 주입 방식은 Spring Framework에서 권장하는 방식으로, 객체의 불변성을 확보할 수 있습니다. final 키워드를 사용할 수 있고, 생성자에 @Autowired를 붙여 의존성을 주입받습니다.

## 상품 도메인 개발
### 상품 엔티티 - 비즈니스 로직 추가
```java
@Entity
@Getter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
public abstract class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

    public void addStock(int quantity) {
        this.stockQuantity += quantity;
    }

    public void removeStock(int quantity) {
        int restStock = this.stockQuantity - quantity;
        if (restStock < 0) {
            throw new NotEnoughStockException("need more stock");
        }
        this.stockQuantity = restStock;
    }
}
```
Item 엔티티는 상품에 대한 정보를 담고 있으며, 재고 수량을 관리하는 비즈니스 로직을 포함하고 있습니다. 이러한 로직은 데이터와 밀접하게 연관되어 있어, 엔티티 내부에 위치시키는 것이 응집력 측면에서 유리합니다.

## 테스트 코드 작성
### JUnit5를 사용한 회원 서비스 테스트
```java
@Test
public void 회원가입() throws Exception {
    // given
    Member member = new Member();
    member.setName("kim");

    // when
    Long saveId = memberService.join(member);

    // then
    assertEquals(member, memberRepository.findOne(saveId));
}
```
위의 테스트 코드는 회원 가입 로직을 검증하는 예제입니다. 이 테스트를 실행하면 예상과는 달리 insert 문이 아닌 select 문이 실행됩니다. 이는 영속성 컨텍스트에서 발생하는 쓰기 지연(Write-Behind) 메커니즘 때문입니다. 트랜잭션이 커밋될 때까지 insert 문이 보류되며, 테스트의 @Transactional은 기본적으로 롤백을 수행하므로 insert 문이 실행되지 않습니다.

### 중복 회원 예외 테스트
JUnit5에서 예외 상황을 테스트하기 위해 assertThrows 메서드를 사용할 수 있습니다.
```java
@Test
public void 중복_회원_예외() throws Exception {
    // given
    Member member1 = new Member();
    member1.setName("kim");

    Member member2 = new Member();
    member2.setName("kim");

    // when & then
    assertThrows(IllegalStateException.class, () -> {
        memberService.join(member1);
        memberService.join(member2);
    });
}
```
위 코드에서 assertThrows를 사용하여 중복 회원 예외가 발생하는지 확인할 수 있습니다.

## 테스트 환경 설정
### 메모리 DB 사용
테스트를 격리된 환경에서 실행하고, 종료 후 데이터를 초기화하는 것이 좋습니다. 이를 위해 메모리 DB를 사용하는 것이 이상적입니다.

test/resources/application.yml 파일을 다음과 같이 설정하여 메모리 DB를 사용할 수 있습니다.

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb # 메모리 데이터베이스를 사용하여 'testdb'라는 데이터베이스를 생성합니다.
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create-drop # 테스트 환경에서 엔티티 상태에 따라 스키마를 생성 및 삭제합니다.
    properties:
      hibernate:
        show_sql: true # SQL 쿼리를 콘솔에 출력합니다.
        format_sql: true # SQL 쿼리를 포맷하여 출력합니다.

logging.level:
  org.hibernate.SQL: debug # SQL 쿼리에 대한 디버그 로깅을 활성화합니다.
```
Spring Boot는 datasource 설정이 없으면 기본적으로 메모리 DB를 사용합니다. 위 설정을 통해 테스트 시에만 메모리 DB를 사용하고, 테스트가 끝나면 자동으로 데이터가 초기화됩니다.

## JPQL Query
JPQL의 from 절에서 사용되는 대상은 SQL의 테이블이 아닌 엔티티입니다. 이 점을 유념하여 쿼리를 작성해야 합니다.

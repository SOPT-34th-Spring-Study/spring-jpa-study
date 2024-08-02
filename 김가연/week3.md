## 애플리케이션 아키텍쳐

### 계층형 구조

![](https://velog.velcdn.com/images/yeoni_/post/844d321d-bc3f-4b5b-8c21-16b77e2f5c33/image.png)


- controller : 웹 계층
- service : 비즈니스 로직, 트랜잭션 처리
- repository : JPA를 직접 사용하는 계층, 엔티티 매니저 사용
- domain : 엔티티가 모여 있는 계층, 모든 계층에서 사용

## 회원 서비스 개발

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


`@Transactional(readOnly=true)` 는 데이터의 변경이 없는 읽기 전용 메서드에 사용하는 어노테이션이다. 영속성 컨텍스트를 flush하지 않으므로 성능적인 면에서 약간의 이점이 존재한다.

### 필드 주입

```java
@Service
@Transactional(readOnly = true)
public class MemberService {
		@Autowired
    private MemberRepository memberRepository;
}
```


현재 코드는 필드 주입(Field Injection) 방식을 사용하고 있다. 필드 주입은 **멤버 객체에 `@Autowired`를 붙여 주입받는 방법**이다. 즉, 주입 받고자 하는 필드 위에`@Autowired` 어노테이션을 붙여주기만 하면 스프링이 직접 의존성을 주입해준다. 하지만 이런 필드 주입은 여러 단점이 존재한다.

먼저 Field Injection을 사용하면 final 제어자를 사용하지 못한다. 즉, 중간에 MemberRepository 객체가 변경될 수도 있다는 뜻이다. 이는 런타임 중에 repository의 불변성을 보장 받을 수 없게 만든다. 또한 테스트 코드를 짜는 데에 있어서도 불편함이 있다. 필드 주입은 달랑 필드 위에 @Autowired 애노테이션이 달려있는 구조이기 때문에, 스프링의 도움을 받지 않으면 객체를 생성하기 어렵다. 따라서 테스트를 하려면 프로젝트 전체를 돌려야 한다. 이는 무거운 프레임워크를 피해 자바 코드만으로 가동하려는 테스트의 의미를 무색하게 만든다.

---

https://velog.io/@sunlake123/의존성-주입의-종류와-필드-주입의-문제점

### 생성자 주입

필드 주입 방식을 생성자 주입 방식으로 바꿔보자.

```java
@Service
@Transactional(readOnly = true)
public class MemberService {
		
    private final MemberRepository memberRepository;
    
    @Autowired // 생성자가 하나면 해당 어노테이션을 생략할 수 있다.
    public MemberService(MemberRepository memberRepository) {
		    this.memberRepository = memberRepository
		}
}
```

생성자 주입은 **생성자에 `@Autowired`을 붙여 의존성을 주입하는 방법으로 객체의 최초 생성 시점에 스프링이 의존성을 주입해준다.** Spring Framework에서 공식적으로 권장하고 있는 방식으로 생성자가 1개 밖에 존재하지 않을 경우 `@Autowired`를 생략할 수 있다. 생성자 주입 방식을 사용하게 되면 필드에 `final` 키워드를 사용할 수 있고 의존성 주입이 최초 1회만 이루어져 **객체의 불변성을 확보**할 수 있다. 특히, `final` 키워드를 추가하면 컴파일 시점에 memberRepository를 설정하지 않은 오류를 체크할 수 있다는 이점이 있다.

위의 코드에서 lombok에서 제공하는 `@RequiredArgsConstructor` 어노테이션을 사용하면 아래와 같이 코드를 줄일 수 있다.

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
}
```

## 회원 기능 테스트
### 영속화(persist)와 쿼리문

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

위는 회원가입 로직을 검증하기 위한 테스트 코드이다. 이때 이 함수를 실행시키면 쿼리문이 어떻게 나갈까?

회원이 추가되었기 때문에 insert문 쿼리가 나갈 것이라는 우리의 예상과는 다르게 insert문 쿼리 없이 select문 쿼리만이 실행된 것을 확인할 수 있다. 그 이유가 무엇일까? 이를 알기 위해서는 영속성 컨텍스트의 동작 과정에 대해서 선제적으로 알아야 한다. 지금부터 살펴보자!

먼저 join 메소드의 코드를 살펴보자.

```java
// MemberService
@Transactional
public Long join(Member member) {
    validateDuplicateMember(member);
    memberRepository.save(member);
    return member.getId();
}
 
// MemberRepository 
public void save(Member member) {
    em.persist(member);
}
```

우리는 새로운 Member 객체를 저장하기 위해 엔티티매니저에 `persist()` 메서드를 통해 영속화하고 있다. 여기서 중요한 점은 단순히 ‘영속화’를 한다는 점이다. 즉, DB에 반영되는 것이 아니다! `persist` 메소드를 통해 객체를 영속화하면 1차 캐시에 저장되는 동시에 쓰기 지연 SQL 저장소에도 SQL 쿼리문이 생성되어 저장된다. persist를 할 때마다 쓰기 지연 SQL 저장소에는 생성된 SQL 쿼리문이 차곡차곡 쌓이게 되고 그 후 트랜잭션이 커밋되면 비로소 DB에 반영 즉, insert문 쿼리가 나가게 되는 것이다.

하지만 테스트 클래스에서 적용되는 `@Transactional` 은 트랜잭션이 `commit` 되지 않고 `rollback`되게 된다. 때문에 쓰기 지연 SQL 저장소에 쌓인 insert 쿼리문이 실행되지 않는 것이다.

### Junit5

강의에서는 테스트코드를 Junit4를 이용해서 아래와 같이 작성하고 있다. 

```java
@Test(expected = IllegalStateException.class)
public void 중복_회원_예외() throws Exception {
   // given
   Member member1 = new Member();
   member1.setName("kim");

   Member member2 = new Member();
   member2.setName("kim");

   // when
   memberService.join(member1);
   memberService.join(member2);
        
   // then
   fail("예외가 발생해야 한다!")
}
```

내가 사용하는 Junit 버전은 5였기 때문에 위의 코드를 따라치자 `Cannot resolve method ‘expected’` 라는 오류가 발생했다. 찾아보니 JUnit 5에서는 `expected` 속성을 지원하지 않아 발생되는 오류였으며 대신에 Junit5에서는 `assertThrows` 메서드를 사용하여 예외가 발생하는지 확인할 수 있다고 한다. 

아래 코드는 강의에서의 코드를 `assertThrows` 메서드를 활용하여 재작성한 코드이다.

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

참고로 처음에 테스트코드에 사용한 함수들에 대한 명확한 이해없이 `assertThrows` 함수 아래에  `fail("예외가 발생해야 한다!")` 을 작성하니 테스트가 실패하는 오류가 발생하였다. 그 이유가 궁금하다면 아래 링크를 참고해보자.  

---

https://dkswnkk.tistory.com/441

https://www.inflearn.com/community/questions/253006/junit5-의-assertions-fail-에-대해-질문이-있습니다

### 테스트 케이스를 위한 설정

테스트는 케이스를 격리된 환경에서 실행하고 테스트가 끝나면 데이터를 초기화하는 것이 좋다. 
그렇다면 테스트를 완전히 격리된 환경에서 할 수 있는 방법은 무엇일까? 바로 메모리 DB를 활용하는 것이다.

![](https://velog.velcdn.com/images/yeoni_/post/dd9b23c9-ad8c-4386-be84-53dce4e9344b/image.png)


위의 사진과 같이 간단하게 테스트용 설정 파일을 추가하여 다르게 사용하면 된다.

main 패키지의 resources 폴더 아래에 있는 application.yml이 바로 설정 파일인데, 이를 test 패키지 아래에도 새롭게 생성하면 되는 것이다.

```yaml
datasource:
	url: jdbc:h2:mem:test
```

datasource의 설정을 위와 같이 작성하면 메모리 DB를 사용한다는 의미가 된다. 다만, 스프링 부트의 경우 datasource 설정이 없으면 기본으로 메모리 DB를 사용하므로 별도의 datasource나 JPA와 관련된 추가 설정을 하지 않아도 된다.

---

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

응집력의 관점에서 볼 때, 데이터를 가지고 있는 쪽에 비즈니스 메소드가 있는 것이 좋다. 때문에 Item 엔티티에 stockQuantity를 활용하는 핵심 비즈니스 메소드를 엔티티에 직접 추가하였다.


### Merge vs Persist

```java
public void save(Item item) {
    if (item.getId() == null) {
        em.persist(item);
    } else {
        em.merge(item);
    }
 }
```

save함수를 동작 로직을 살펴보면, id가 없을 경우 persist()를 실행하고 있을 경우, 데이터베이스에 저장된 엔티티를 수정하는 merge()를 실행한다. 여기서 persist()와 merge()의 차이점이 무엇일까?

Merge는 Detached 상태의 Entity를 다시 영속화 하는데 사용되고 Persist는 최초 생성된 Entity를 영속화하는데 사용된다. 지금은 그냥 간단하게 persist는 비영속 상태를 영속 상태로, merge는 준영속 상태를 영속 상태로 바꿔준다고만 알아두자. → 더 자세한 내용은 섹션 7, 웹 계층에서 다룬다. 
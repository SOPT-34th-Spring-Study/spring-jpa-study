# 4️⃣ 회원 도메인 개발
## 회원 리포지토리 개발

## 영속성 컨텍스트(Persistence Context)

> Entity를 영구적으로 저장할 수 있는 환경
> 

JPA에서는 `EntityManager` 로 Entity를 `영속성 컨텍스트`에서 관리한다.

```java
@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;
}
```

## JPQL과 SQL의 차이점

JPQL은 ‘엔티티 객체’를 대상으로 쿼리, SQL은 ‘테이블’ 대상으로 쿼리

## 회원 서비스 개발

## `@Transactional(readOnly = true)`

> 데이터의 변경이 없는 읽기 전용 메서드에는 가급적이면 `readOnly = true` 를 사용하자!
> 

👉🏻 **이점**

- 트랜잭션 commit 시 영속성 컨텍스트가 자동으로 flush되지 않으므로 조회용으로 가져온 Entity의 예상치 못한 수정을 방지할 수 있다.
- JPA는 해당 트랜잭션 내에서 조회하는 Entity는 단순 조회용임을 인식하고, 변경 감지를 위한 Snapshot을 따로 보관하지 않으므로 메모리가 절약되는 성능상 이점이 존재한다.

**👉🏻 사용 Tip**

클래스 전체에 readOnly = true를 걸어둔 다음, 회원 가입 join 메서드에만 @Transactional 걸어주기. join에서는 readOnly = false 가 우선이 됨!

```java
Service
@Transactional(readOnly = true)
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    // 회원 가입
    @Transactional
    public Long join(Member member) {
        validateDuplicateMember(member); // 중복 회원 검증
        memberRepository.save(member);
        return member.getId();
    }
    
    // 회원 전체 조회
      public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    private Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
```

## `@Column(unique = true)`

> 멀티스레드 환경 :: 유니크 제약 조건
> 

MemberService의 검증 로직을 살펴보자.

```java
    private void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }
```

이 코드의 경우엔 A와 B의 두 개의 스레드가 동시에 “John”이라는 이름으로 회원 등록을 시도할 경우, 두 스레드 모두 `validateDuplicateMember`를 호출하여 회원명 "John"에 대해 조회하고 아직 "John"이라는 회원이 없으므로 통과하게 된다. → 유니크 제약 조건 위반!

따라서 실무에서는 멀티 쓰레드 상황을 고려해서 Member 테이블의 name 컬럼에 `Column(unique = true)` 제약 조건을 추가하는 것이 안전하다.

## 생성자 주입을 사용하자!

**필드**

```java
@Service
@Transactional(readOnly = true)
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;
}
```

**Setter 메서드**

```java

@Service
@Transactional(readOnly = true)
public class MemberService {

    private MemberRepository memberRepository;
    
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

⭐ **생성자** ⭐

```java
@Service
@Transactional(readOnly = true)
public class MemberService {

    private MemberRepository memberRepository;

    @Autowired //생성자가 1개만 있는 경우 생략 가능
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

lombok 적용해서 **`@RequiredArgsConstructor`** 사용하면 final만 있는 필드를 가지고 생성자 자동 생성해줌.

```java
@Service
@Transactional(readOnly = true)
**@RequiredArgsConstructor**
public class MemberService {

    private **final** MemberRepository memberRepository;
```

## 회원 기능 테스트

## JUnit5

`@Runwith` 은 JUnit4에서 사용되고, JUnit5로 넘어오면서 `@ExtendWith`으로 바뀌었다. 그치만 이 또한 **`@SpringBootTest`** 어노테이션을 사용하면 생략 가능하다!

## 테스트에서 트랜잭션의 Rollbcak

회원가입을 검증하기 위한 아래의 테스트 메서드를 실행시킨 후, 쿼리문을 살펴보면 insert 쿼리가 발생하지 않는 것을 확인할 수 있다. 이유가 뭘까?

```java
    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long saveId = memberService.join(member);

        //then
        assertEquals(member, memberRepository.findOne(saveId));
    }
```
![image](https://github.com/user-attachments/assets/24b0870c-1f8a-460f-a8c9-0dd3546f445b)


- `@Transactional` 이 Test 케이스에 적용되면, 각각의 테스트를 실행할 때마다 트랜잭션을 시작하고 **테스트가 끝나면 트랜잭션을 강제로 롤백** 된다. 이는 테스트가 DB를 변경하지 않도록 하기 위함이다!
- `persist` 메서드를 호출한다고 해서 `insert` 쿼리가 실행되는 것은 아니다. insert 쿼리는 트랜잭션이 commit 되는 순간 `flush`가 되면서 실행된다.

→ 즉, 트랜잭션이 commit 되는 시점에 flush가 발생해서 DB에 변경사항이 반영되는데, 스프링의 테스트에서는 @Transactional 이 있으면 트랜잭션이 종료되면서 롤백을 해버리기 때문에 insert 쿼리가 DB에 반영되지 않는 것이다.

## 중복 회원 검증

```java
    @Test
    public void 중복_회원_예외() throws Exception {
        // given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");

        // when & then
        **assertThrows(IllegalStateException.class, () -> {
            memberService.join(member1);
            memberService.join(member2);
        });**
    }
```

## 운영과 테스트의 DB의 분리

1. test 환경에도 main과 똑같은 application.yml을 둔다. (복사)

![image](https://github.com/user-attachments/assets/71e0bf9d-92af-4b5b-aa12-639c5485200b)


2. test의 yml의 내용(url)을 아래와 같이 바꾼다.

h2를 내려도 test가 정상적으로 잘 실행되는 것을 확인할 수 있다.

```yaml
spring:
  datasource:
   url: jdbc:h2:mem:test
```

❗But, yml 안에 별도의 설정 내용이 없으면 spring boot가 기본적으로 메모리 모드로 실행하므로 사실상 빈 내용이어도 상관없긴 하다!

---

# 5️⃣ 상품 도메인 개발


## 상품 엔티티 개발(비즈니스 로직 추가)

데이터를 가지고 있는 쪽에 비즈니스 메서드가 있는 것이 좋다. 그래서 stockQuantity 필드를 사용하는 재고 추가, 재고 삭제 메서드를 Item 엔티티에 작성해 주었다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@Getter @Setter
@DiscriminatorColumn(name = "dtype")
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    **private int stockQuantity;**

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();

    //==비즈니스 로직==//
    public void addStock(int quantity) {
        this.**stockQuantity** += quantity;
    }

    private void removeStock(int quantity) {
        int restStock = this.stockQuantity - quantity;
        if(restStock < 0) {
            throw new NotEnoughStockException("need more stock");
        }
        this.**stockQuantity** = restStock;
    }
}

```

## 상품 리포지토리 개발

## Persist 와 Merge

JPA에서 persist와 merge는 엔티티를 DB에 저장하거나 업데이트 할 때 사용하는 메서드이다. 자세한건 섹션 7의 ‘웹 계층 개발’에서 다룰 예정이니 간단하게만 짚고 넘어가도록 하자.

```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {
    '''
    
    public void save(Item item) {
        if (item.getId() == null) {
            em.persist(item);
        } else { 
            em.merge(item); 
        }
    }
    
    '''
}
```

`persist`는 새로운 엔티티를 저장할 때 사용하고, 트랜잭션 커밋 시점에 ‘INSERT’ 쿼리가 실행된다. `merge`는 기존에 존재하던 엔티티를 업데이트하고, 트랜잭션 커밋 시점에 ‘UPDATE’ 쿼리가 실행된다.

## 상품 서비스 개발

ItemService 는 ItemRepository 에 단순히 위임만 하는 클래스이다.

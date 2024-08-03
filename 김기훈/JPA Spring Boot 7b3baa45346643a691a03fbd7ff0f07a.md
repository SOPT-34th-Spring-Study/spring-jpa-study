# JPA Spring Boot

# 2주차

# 도메인 모델 및 테이블 설계

![Untitled](JPA%20Spring%20Boot%207b3baa45346643a691a03fbd7ff0f07a/Untitled.png)

- 이것이 도메인 모델이긴함
- 이것을 가지고 실제 Entity를 만들자

![Untitled](JPA%20Spring%20Boot%207b3baa45346643a691a03fbd7ff0f07a/Untitled%201.png)

- 다양한 Entity들간으 관계를 보여준것 뿐 실제로 이렇게 사용하지는 않음

![Untitled](JPA%20Spring%20Boot%207b3baa45346643a691a03fbd7ff0f07a/Untitled%202.png)

### 관례

- 대부분 소문자 + _ 타입으로 지정 ,snake case

## 연관관계 주인 정하기

- 무조건 “많은쪽” 이 주인임

## Entity 클래스 개발

- Getter는 가능
- Setter는 지양 , 필요한 경우에만 붙임

### 주의 :

양방향 관계에서는 연관관계 주인을 설정해줘야햠 

왜? 

양쪽에서의 엔티티 변형이 일어날수 있기 때문인데 주인을 기준으로 update된것을 파악해야함

## Entity 기본

- 기본생성자 필요함

그리고 Spring 시작시 DDL Auto로 설정한 script를 가지고 수정해서 시작시에 작동하는 .sql script를 작성하는 것이 좋음

# Entity 설계시 주의점

- Setter 사용하지 말기
    - 가급적이면 !
- 모든 연관관계는 지연로딩으로 설정!
    - EAGER는 예측이 어렵고 어떤 SQL이 실행될지 추적 어려움
    - JPQL을 실행할때 N+1문제가 자주 발생함
    - 실무에서는 모두 LAZY로 실행함
    - 연관 entity를 실무에서 조회하게 되면 fetch join 혹은  엔티티 그래프 기능 사용
    - @XtoOne 은 기본이 즉시로딩이므로 지연로딩으로 설정해야함
    - 
- 컬렉션은 필드에서 초기화하자
    - 컬렉션은 필드에서 바로초기화가 안전함
    - List<Order> = new ArrayList();
    - Null 문제 안전함
    - 하이버네이트에서 영속화할때 컬렉션감쌈 → PersistentBag 컬렉션으로 변경됨
        - 따라서 필드초기화로 바로 초기화해주는것이 안전함

## 테이블 , 컬럼 생성 전략

- 테이블명 알아서 snake case 로 바꿈
    - 대문자 → 소문자

## Cascade 전략

- 영속화 전파

## 연관 메소드

![Untitled](JPA%20Spring%20Boot%207b3baa45346643a691a03fbd7ff0f07a/Untitled%203.png)
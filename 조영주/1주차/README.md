# 섹션 1

생성일: 2024년 7월 2일 오후 7:35

# 섹션1

참고로, 이번 강의에서는 Spring Data JPA에 대해서 설명하지는 않고, 

Spring Boot와 JPA를 가지고 어떻게 애플리케이션을 잘 만드는지를 배울 것!

## 프로젝트 생성

---

1. 해당 의존성들을 추가하고 프로젝트를 생성한다.

![image](https://github.com/choyeongju/spring-jpa-study/assets/128598386/cad210dd-ea6d-4e34-9541-08912c8156bf)


1. Lombok을 설치했으면, Annotation Processors에 들어가서 Enable annotation processing을 활성화 해준다.

![image](https://github.com/choyeongju/spring-jpa-study/assets/128598386/9ae17000-b006-49b5-b7b6-6db98088da91)

💡 단축키 꿀팁

`option + command + v` 하면 변수 자동완성 시켜준다
![image](https://github.com/choyeongju/spring-jpa-study/assets/128598386/174128b2-3c76-43d2-8421-c70fc9e4e1ac)


### Lombok?

어노테이션 기반으로 코드 자동완성 기능을 제공하는 라이브러리.

| 어노테이션           | 설명                                                                                  |
|----------------------|---------------------------------------------------------------------------------------|
| @Getter              | code가 컴파일 될 때 getter 메서드들을 생성한다.                                       |
| @Setter              | code가 컴파일 될 때 setter 메서드들을 생성한다.                                       |
| @ToString            | toString() 메서드를 생성한다.                                                         |
| @Data                | @Getter(모든속성), @Setter(final이 붙지 않은), @ToString, @EqualsAndHashCode, @RequiredArgsConstructor 위의 어노테이션들을 합쳐둔 어노테이션이다. |
| @NoArgsConstructor   | 파라미터(매개변수)가 없는 생성자를 생성한다.                                          |
| @RequiredArgsConstructor | final, @NonNull이 있는 필드를 포함하여 생성자를 생성한다.                           |
| @AllArgsConstructor  | 모든 필드를 파라미터(매개변수)로 갖는 생성자를 생성한다.                              |
| @Builder             | 해당 클래스에 빌더 패턴을 사용할 수 있도록 해준다.                                    |
| @Value               | 불변 클래스를(Immutable Class) 생성해준다. 모든 필드를 Private, Final 로 설정하고, Setter를 생성하지 않는다.(상수로 만들어준다.) |
| @NonNull             | 필드의 값이 null이 될 수 없음을 명시해준다.                                           |
| @Slf4J               | Slf4J 설정을 이용하여 로그 기능 사용할 수 있다. 

## 라이브러리 살펴보기

---

## 커넥션 풀 - HikariCP

JDBC API를 사용하여 데이터베이스와 연결하기 위해 Connection 객체를 생성하는 작업은 비용이 굉장히 많이 드는 작업 중 하나입니다. 따라서 웹 애플리케이션과 같은 다중 사용자 환경에서 데이터베이스 연결을 효율적으로 관리하기 위해 커넥션 풀이 사용됩니다.

1. 클라이언트와 웹 애플리케이션에서 데이터베이스 연결을 위해 미리 일정수의 connection 객체를 만들어 Pool에 담아 둔다.
2. 추후 사용자의 요청이 발생하면 Pool에서 생성되어 있는 Connection 객체를 넘겨준다.
3. 처리가 끝나면 실행된 상태로 connection 객체를 다시 Pool에 반환하여 보관

[DBCP (DB connection pool)의 개념부터 설정 방법까지! hikariCP와 MySQL을 예제로 설명합니다! 이거 잘 모르면 힘들..](https://www.youtube.com/watch?v=zowzVqx3MQ4)

## Thymeleaf

'**템플릿 엔진**'의 일종으로, html 태그에 속성을 추가해 페이지에 동적으로 값을 추가하거나 처리할 수 있습니다.

## 로깅 Logging

### Logging 라이브러리가 필요한 이유?

디버깅 목적으로 `System.out.println` 과 같은 출력문을 사용해 본 적이 있지 않나요?! 하지만 이런 방법은 많은 불편함이 존재합니다. 

### System.out.println의 문제점

**1. 로그 레벨 설정 불가**

 로그 라이브러리는 DEBUG, INFO, WARN, ERROR 등의 로그 레벨을 사용하여 원하는 레벨 이상의 로그만 출력할 수 있습니다. 하지만 `System.out.println`은 모든 출력이 동일한 레벨로 취급되므로, 불필요한 디버깅 메시지까지 모두 출력하게 되어 로그 관리가 어려워집니다.

| TRACE | 가장 상세한 로그 레벨, 애플리케이션의 실행 흐름과 디버깅 정보를 좀 더 상세하게 기록 |
| --- | --- |
| DEBUG |  개발 단계에서 디버그 용도로 사용하는 메시지 |
| INFO | 상태 변경과 같은 정보성 메세지 |
| WARN |  프로그램 실행에는 문제 없으나, 향후 시스템 에러의 원인이 될 수 있는 경고성 메세지 |
| ERROR | 심각한 문제 또는 예외 상황을 나타내며, 애플리케이션의 정상적인 동작에 영향을 미칠 수 있는 문제 |
| FATAL | 애플리케이션의 동작을 중단시킬 수 있는 아주 심각한 에러(복구 불가능/어려움)가 발생한 상태 |

**2. 출력 관리의 어려움**

 로그 라이브러리는 로그를 파일이나 외부 서버에 저장할 수 있습니다. 하지만 `System.out.println`은 기본적으로 콘솔에만 출력을 하기 때문에, 이러한 출력을 파일로 기록하거나 관리하기 어렵습니다.

**3. 유지보수의 어려움**

 개발 중에 추가한 `System.out.println`은 배포 전 주석 처리하거나 제거해야 합니다. 이를 자동화하는 도구가 있기는 하지만, 여전히 개발자가 일일이 신경 써야 하는 부분이 있습니다. 반면, 로그 라이브러리는 로깅 설정만 변경하면 쉽게 로깅 레벨을 조정할 수 있어 유지보수가 쉬워집니다.

### SLF4J(Simple Logging Facade for Java)

> java.util.logging, logback, log4j와 같은 다양한 로깅 프레임워크에 대한 
**인터페이스 역할**을 하는 라이브러리
> 
- logging 관련 라이브러리는 다양한데, 이러한 **라이브러리들을 하나의 통일된 방식으로 사용할 수 있**는 방법을 제공합니다. 즉, 애플리케이션은 SLF4J를 사용함으로써, 어떠한 logging 라이브러리를 사용하든 같은 방법으로 로그를 남길 수 있습니다.
- 로크 라이브러리를 교체하더라도, 어플리케이션의 코드는 변경될 필요가 없습니다.
- SLF4J는 추상 클래스이기 때문에 단독으로 사용할 수 없습니다.

(동작과정 참고)

[[Logging] SLF4J란?](https://livenow14.tistory.com/63)

### LogBack

> SLF4j 의 구현체
> 
- Logback은 스프링 부트의 기본으로 설정되어 있어서 사용시 별도의 라이브러리를 추가하지 않아도 됩니다. spring-boot-starter-web 안에 spring-boot-starter-logging에 구현체가 있습니다.
- SLF4J를 사용하여 로깅 처리를 하게 되면, 실제 로그는 LogBack에서 출력하게 됩니다.
- 설정 파일 읽어가는 순서
:: resources 밑의 logback-spring.xml 파일 → 없으면 .yml파일 → 둘다 있으면 yml 먼저 읽고 xml

### `@Slf4j` 사용 방법

> Lombok 라이브러리를 사용하여 자동으로 SLF4J Logger를 생성해주는 애노테이션
> 

1. application.yml 설정
    
    ```yaml
    logging:
      file:
        name: ${user.dir}/log/test.log  # 로깅 파일 위치이다.
        max-history: 7 # 로그 파일 삭제 주기이다. 7일 이후 로그는 삭제한다.
        max-size: 10MB  # 로그 파일 하나당 최대 파일 사이즈이다.
      level:  # 각 package 별로 로깅 레벨을 지정할 수 있다.
        com.project.study : error
        com.project.study.controller : debug
    ```
    
    (ex) Spring Boot 애플리케이션을 실행하면, 로그 파일이 아래와 같은 경로에 생성됩니다:
    
    - 애플리케이션 실행 디렉토리: `/home/user/myapp`
    - 로그 파일 경로: `/home/user/myapp/log/test.log`
    
    💡(참고) **`application.properties` 방식** 으로 설정할 수도 있으나, 이는 설정이 단순하고 간단한 프로젝트에 적합하고, **`application.yml` 방식**은 복잡한 설정이나 중첩된 설정을 필요로 하는 프로젝트에 적합합니다.
    
    ```yaml
    # 전체 로그 레벨 설정(기본 info)
    logging.level.root=info
    # hello.springmvc 패키지와 그 하위 로그 레벨 설정
    logging.level.hello.springmvc=trace
    ```
    
2. 기본적으로 Spring Boot는 Logback을 사용하지만, 추가적인 Logback 설정이 필요할 경우 `src/main/resources` 디렉토리에 `logback-spring.xml` 파일을 생성하여 설정을 정의할 수 있습니다. 
이 설정 파일은 `application.yml` 파일의 설정을 좀 더 세밀하게 조정할 수 있습니다.
    - Logback을 이용하여 로깅을 수행하기 위해서 필요한 주요 설정요소로는 Logger, Appender, Encoder가 있습니다.
    - 참고 (저는 잘 모르겠음~..ㅎ)
        
        [[Logging] Logback이란?](https://livenow14.tistory.com/64)
        
        [[Java] Log..그리고 slf4j와 logback](https://velog.io/@nias0327/Java-Log..그리고-slf4j와-logback)
        

1. Gradle에 lombok 라이브러리 추가
    
```yaml
dependencies {
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```
    

4. 실제 사용
    - `@Slf4j` 을 사용하지 않으면, SLF4J Logger를 직접 생성하여 사용해야 합니다.
        
        ```java
        import org.slf4j.Logger;
        import org.slf4j.LoggerFactory;
        
        public class MyClass {
            **private static final Logger logger = LoggerFactory.getLogger(MyClass.class);**
        
            public void NoneSlf4jExample() {
                logger.debug("Debugging message");
                logger.info("Info message");
                logger.warn("Warning message");
                logger.error("Error message");
            }
        }
        ```
        
    - `@Slf4j` 을 사용하면, Lombok이 자동으로 SLF4J Logger를 생성해줍니다.
        
        ```dhall
        import lombok.extern.slf4j.Slf4j;
        
        **@Slf4j**
        public class MyClass {
            public void Slf4jexample() {
                log.debug("Debugging message");
                log.info("Info message");
                log.warn("Warning message");
                log.error("Error message");
            }
        }
        ```
        

## View 환경 설정

---

```java
@Controller
public class HelloController {

    @GetMapping("hello")
    public String hello(Model model) {
        model.addAttribute("data", "hello!!!");
        return "hello"; 
    }
}
```

### @Controller

- 해당 클래스를 Spring MVC에서 컨트롤러로 사용하겠다는 것을 Spring에게 알려줍니다.

### org.springframework.ui.Model

> Model은 HashMap 형태를 갖고 있으며, <Key,Value>값을 가지고 있습니다.
> 
- 모델(Model)은 컨트롤러에서 데이터를 뷰(View)로 전달할 때 사용됩니다.
- `addAttribute()`와 같은 기능(Map의 put과 같은 기능)을 통해 모델에 원하는 속성과 그것에 대한 값을 주어 전달할 뷰에 데이터를 전달할 수 있습니다.
- 참고하면 좋을 링크
    
    [[Spring] Model 객체](https://velog.io/@msriver/Spring-Model-객체)
    

### return "hello";

- `hello` 라는 이름의 뷰를 반환합니다.
- Spring MVC 설정에 따라 `hello`라는 이름의 뷰 파일을 찾아서 랜더링합니다. 아래의 `hello.html` 파일이 될 수 있습니다.
**랜더링 : 서버로부터 HTML 파일을 받아 브라우저에 뿌려주는 과정*
    
![image](https://github.com/choyeongju/spring-jpa-study/assets/128598386/324682dd-434b-4f8f-a318-61a092871973)


    

hello.html 파일의 내용은 아래와 같습니다.

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
**<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>**
</body>
</html>
```

- 이 때, 순수한 html 랜더링을 하면 화면에 `안녕하세요. 손님` 이 나타나게 됩니다.
- spring boot를 실행시켜서 SSR(Server Side Rendering) 을 하게 되면 data안에 담겨있던 hello!!!로 바뀌어서 화면에 나타나는 것을 확인할 수 있습니다.
    
![image](https://github.com/choyeongju/spring-jpa-study/assets/128598386/ad136bce-bb6a-4131-994a-bd022ef55592)

    

→ 이는 스프링 부트의 thymleaf가 viewName 매핑을 해주기 때문에 가능합니다.
`resources:templates/ +{ViewName}+ .html` 

## **H2 데이터베이스 설치**

---

### h2 데이터베이스 실행 방법

1. cd h2 → cd bin
2. (Mac) `./h2.sh`

### 데이터베이스 파일 생성 방법

1. 데이터베이스 파일 생성 (한 번만)
: `jdbc:h2:~/jpashop` 이 URL을 사용하여 H2 데이터베이스에 접속하면, 사용자의 홈 디렉토리(~) 에~/jpashop.mv.db 파일이 생성됩니다.

터미널에 cd ~ , ls -arlth 를 차례로 입력하여 jpashop.mv.db 파일( H2 데이터베이스의 실제 데이터가 저장되는 곳)이 생성 되었는지 확인합니다.
2. 서버 모드로 전환
: 위에서 데이터베이스 파일의 생성을 확인했으면, 이제부터는 `jdbc:h2:tcp://localhost/~/jpashop` 이 URL을 사용하여 데이터베이스에 접근합니다. 이 모드에서는 H2 데이터베이스가 TCP 서버로 동작하게 됩니다.

## **JPA와 DB 설정, 동작확인**

---

### application.yml 설정

```yaml
spring:
  datasource:
   url: jdbc:h2:tcp://localhost/~/jpashop;MVCC=TRUE
   username: sa
   password:
   driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        show_sql: true
        format_sql: true

logging:
  level:
    org.hibernate.sql: debug
```

**ddl-auto 속성**

ddl-auto란, 엔티티만 등록해 놓으면 DDL(Dage Definition Language)를 자동으로 작성하여 테이블을 생성하거나 수정해주는 설정이다.
| **옵션** | **설명** |
| --- | --- |
| none(default) | 속성이 존재하는 것이 아니라 위의 4가지 경우를 제외한 모든 경우에 해당한다. <br>→ 아무 일도 일어나지 않는다. |
| validate | 애플리케이션 실행 시점에 엔티티와 테이블이 정상 매핑 되었는지만 확인한다. <br>→ 만약 테이블이 아예 존재하지 않거나, 테이블에 엔티티의 필드에 매핑되는 컬럼이 존재하지 않으면 예외를 발생시키면서 애플리케이션을 종료한다. |
| update | 애플리케이션 실행 시점에 새로운 컬럼이 추가되는 변경사항만 반영한다. <br>→ 어떠한 엔티티 클래스의 String 필드를 Int로 변경하더라도, 해당 엔티티에 매핑되는 테이블의 해당 컬럼은 숫자 타입으로 바뀌지 않고 문자열 타입으로 유지된다. |
| create | 애플리케이션 실행 시점에 기존 테이블들을 모두 삭제하고, 다시 생성한다. |
| create-drop | 애플리케이션 종료 시점에 기존 테이블들을 모두 삭제하고, 다시 생성한다. |



**상황별 ddl-auto 속성의 사용 방법**

마냥 편리한 속성만이라고 생각하면 오산! 잘못 다루면 돌이킬 수 없는 결과를 가져오는 아주 위험한 기능입니다..

| 개발 초기, 로컬에서 테스트 | create 또는 update |
| --- | --- |
| 테스트 서버 | update 또는 validate |
| 스테이징 서버 | validate 또는 none |
| 운영 서버 | create, create-drop, update는 절대 사용하면 안된다!!!! validate나 none을 사용한다. |

운영 서버에서 만약 `create`나 `create-drop`으로 설정한 채로 운영 DB에 연결하여 애플리케이션을 실행하면, 운영 DB의 테이블이 몽땅 삭제되는 대참사가 일어날 수 있다 . `update`의 경우에도 만약 새로 추가된 컬럼이 not null 이라면 해당 변경사항이 반영되지 않은 버전을 배포할 시에 테이블이 INSERT 되지 않을 수 있다.

관련하여 아래 영상 재밌게 봤으니까 다들 시간나면 봐보셔용 ㅎㅎ

[재난급 서버 장애내고 개발자 인생 끝날뻔 한 썰 - 납량특집! DB에 테이블이 어디로 갔지?](https://www.youtube.com/watch?v=SWZcrdmmLEU)

## MemberRepository 테스트(JUnit 5)

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;

    @Test
    @Transactional
    @Rollback(false)
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long saveId = memberRepository.save(member);
        Member findMember = memberRepository.find(saveId);

        //then
        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
    }
}
```

- `@ExtendWith(SpringExtension.class)` 는 JUnit 4의 `@RunWith(SpringRunner.class)`와 동일
- `@Transactional` : 없으면 테스트 에러 뜬다
- `@Rollback(false)` : 기본적으로 테스트를 진행하고 난 후에는 db를 다시 롤백 시키는데, 이 어노테이션 붙여준 다음 테스트를 실행하면, 테스트 할 때 사용했던 정보가 db에 저장되어 있다.
- jar 빌드로 동작 확인하는 방법
    - ./gradlew clean build → build.libs 경로로 이동 → java -jar jpashop-0.0.1-SNAPSHOT.jar

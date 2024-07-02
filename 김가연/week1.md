### 무엇을 알게 되었나요?

LogBack -> SLF4J 인터페이스의 구현체
### application.yml
```
  jpa:
    properties:
      hibernate:
#        show_sql: true

logging.level:
  org.hibernate.SQL: debug
#  org.hibernate.type: trace
```
`show_sql: true`의 경우, sql문은 System.out.println으로 출력하는 것 -> 사용하면 안됨
반면` org.hibernate.SQL: debug`은 로그로 출력한다. -> 운영환경에서는 모두 로그를 통해서 출력해야함

### @Transactional
`@Transactional` 어노테이션이 test code에 있을 경우, 테스트가 종료되면 자동으로 Rollback을 수행하여 db에 데이터가 존재하지 않는다. -> 반복적인 테스트 수행을 위해 Rollback 
만약 테스트를 수행하고 db에 데이터가 담겨있는 것을 확인하고 싶다면? -> `@Rollback(false)` 어노테이션 사용

### Dependency
쿼리 파라미터를 ?가 아니라 실제로 보고 싶다면 아래 의존성을 추가한다.
```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.6.2'
```
다만, 쿼리 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하기 때문에 운영 시스템에 적용하기 위해서는 성능 테스트를 진행 후 사용하는 것이 좋다.

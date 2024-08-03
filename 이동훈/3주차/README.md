# JPQL Query

- `from`의 대상이 테이블이 아니라 **엔티티**이다.

# Test Code

- 기본적으로 테스트 코드에서 `@Transactional`은 Rollback을 시킨다.
- 즉, 영속성 컨텍스트의 내용을 flush하지 않는다.
- 롤백시키고 싶지 않다면 `@Rollback(false)`를 해주면 된다.

# 테스트용 `application.yml`

- 테스트는 케이스가 격리된 환경에서 실행되고, 끝나면 데이터를 초기화하는 것이 좋다.
- 그런 면에서 메모리 DB를 사용하는 것이 가장 이상적이다.
- 따라서 다음과 같이 `application.yml`을 생성하면 서비스 DB와 격리된 환경으로 테스트 코드를 테스트해볼 수 있다.

## `test/resources/application.yml`

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

  - 스프링 부트는 datasource 설정이 없으면, 기본적으로 메모리 DB를 사용하고 , driver-class도 현재 등록된 라이브러리를 보고 찾아준다.
  - 추가로 `ddl-auto` 도 `create-drop` 모드로 동작한다. 따라서 데이터소스나, JPA 관련된 별도의 추 가 설정을 하지 않아도 된다.
<aside>
💡 범위 : ~ 상품 도메인 개발

</aside>

## 1. 프로젝트 설정

- 프로젝트 생성
    - *3.0.5버전은 java 17에서 지원, java11 지원 안 함.

![image](https://github.com/SOPT-34th-Spring-Study/spring-jpa-study/assets/63058347/66ab81ec-8afb-4e7f-9d9a-704f84f5582b)

- 라이브러리 살펴보기
    - hikariCP 커넥션 풀 → 찾아보기
    
- 라이브러리 추가(from. 강의자료)
    - Validation (JSR-303 validation with Hibernate validator) 모듈을 꼭! 추가해주세요.(영상에
        
        없습니다.)
        
        - build.gradle에 다음 코드 추가
            
            ```python
            implementation 'org.springframework.boot:spring-boot-starter-validation'
            ```
            
    
    - JUnit4 추가 (안 하면 JUnit5로 동작)
        
        ```python
        testImplementation("org.junit.vintage:junit-vintage-engine") {
        		exclude group: "org.hamcrest", module: "hamcrest-core"
        	}
        ```
        
    - 실행시 Gradle →  IntelliJ IDEA로 변경(속도때문)
        
        **Preferences** Build, Execution, Deployment Build Tools Gradle
        Build and run using: Gradle → IntelliJ IDEA
        Run tests using: Gradle → IntelliJ IDEA
        
    - html 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경
        - 인텔리J 컴파일 방법: 메뉴 build Recompile
        
        ```python
        implementation 'org.springframework.boot:spring-boot-devtools'
        ```
        

- JPA와 DB 설정, 동작확인
    - Member 엔티티 생성
    - MemberRepository 생성
        - 🤔`@PersistenceContext` 란?
            - EntityManager를 빈으로 주입할 때 사용하는 어노테이션
    - MemberRepositoryTest생성
        - `@RunWith(SpringRunner.class)` JUnit4에는 반드시 필요함!
        - `@Rollback(false)` → 바로 commit → h2데이터베이스 바로 반영되어 확인가능
        

## ❗️TIP - 쿼리 파리미터 로그 남기기

```python
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'
```

![image](https://github.com/SOPT-34th-Spring-Study/spring-jpa-study/assets/63058347/32161479-deac-4063-8876-5ed03532328a)

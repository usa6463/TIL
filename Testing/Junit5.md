# Junit5

- JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage
    - JUnit Platform: 테스팅 프레임워크를 실행하기 위한 기반 제공.
        - TestEngine API 정의
          - platform 위에서 실행되는 테스팅 프레임워크를 개발하는데 사용
        - Console Launcher 제공
          - CLI로 플랫폼 실행
        - JUnit Platform Suite Engine 제공
          - 플랫폼에서 1개 이상의 test engine을 사용하여 custom test suite 실행하는데 사용.
        
    - JUnit Jupiter 
        - 테스트 작성을 위한 신규 programming model과 확장을 위한 extension model의 조합.
        - Jupiter 기반의 테스트를 platform 위에서 돌리기 위한 TestEngine을 제공한다. 
    
    - JUnit Vintage
        - JUnit3, JUnit4 기반의 테스트를 실행하기 위한 TestEngine을 제공
    

- Meta-Annotations and Composed Annotations
    - Jupiter annotation은 meta-annotation으로 사용될 수 있다.  
    - 사용자만의 합성(Composed) 어노테이션을 생성할 수 있다.
    - example
        -     @Target(ElementType.METHOD)
              @Retention(RetentionPolicy.RUNTIME)
              @Tag("fast")
              @Test
              public @interface FastTest {
              }
        -     @FastTest
              void myFastTest() {
              // ...
              }
        - @FastTest라는 Composed Annotation을 사용하는걸로 @Tag("fast"), @Test 등을 붙일 수 있다.


- Test Classes and Methods
    - Test Class: 최소 1개의 test method를 가진 any top-level class, static member class, @Nested Class
    - Test Method: 아래 어노테이션으로 directly annotated 되거나 meta-annotated된 인스턴스 메소드
        - @Test, @RepeatedTest, @ParameterizedTest, @TestFactory, @TestTemplate
    - Lifecycle Method: 아래 어노테이션으로 directly annotated 되거나 meta-annotated된 인스턴스 메소드
        - @BeforeAll: 가장먼저 실행. 한번만 실행.
        - @BeforeEach: @BeforeAll 다음으로 실행되고, 각 test method 실행전에 실행됨.
        - @AfterEach: 각 test method 실행후에 실행됨.
        - @AfterAll: 가장 마지막에 실행. 한번만 실행. 
        - 
    - 접근제어자(Access Modifier) 관련
        - public이 될 필요는 없지만, private은 되면 안된다.
        - 특별히 기술적인 이유가 없는한 public을 생략한다.
            - public을 붙여야할 상황
                - 테스트 클래스가 다른 패키지에 있는 테스트 클래스를 상속하는 경우
                - Java Module System을 사용할 때 module path에 대한 testing을 단순화 하고자 할 때.


- Display Name Generator
    - 클래스에 @DisplayNameGeneration 붙이는 걸로 해당 클래스내 test method의 display name 값 생성이 가능
    - @DisplayName이 붙어 있으면 해당 display name이 우선된다.
    

- Default Display Name Generator
    - ```src/test/resources/junit-platform.properties``` 경로에서 ```junit.jupiter.displayname.generator.default``` 값 설정하는 것으로 Default Display Name Generator 를 세팅할 수 있다.
        - example
            -     junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
    - 기본적으로 ```junit.jupiter.displayname.generator.default```은 org.junit.jupiter.api.DisplayNameGenerator.Standard를 사용.
    - test method들은 기본적으로 @DisplayName, @DisplayNameGeneration가 명시되어 있지 않다면 ```junit.jupiter.displayname.generator.default```의 세팅을 사용한다.


- Display Name 적용 순서
    1. @DisplayName 값
    2. @DisplayNameGeneration에 명시된 DisplayNameGenerator
    3. junit.jupiter.displayname.generator.default에 명시된 세팅
    4. org.junit.jupiter.api.DisplayNameGenerator.Standard
    

## 참고
- [문서] [**JUnit 5 User Guide**](https://junit.org/junit5/docs/current/user-guide/)
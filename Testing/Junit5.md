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
    


## 참고
- [문서] [**JUnit 5 User Guide**](https://junit.org/junit5/docs/current/user-guide/)
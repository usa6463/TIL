## 객체(Object)
- 싱글턴 (하나 이상의 인스턴스를 가질 수 없는 형태의 클래스)
- 객체는 실행중인 JVM에서 최초로 접근할 때 자동으로 인스턴스화 된다.즉 객체에 접근하기 전 까진 인스턴스화되지 않는다.
- 객체는 다른 클래스를 상속 가능하지만, 객체로부터 상속하여 클래스를 만들 수 없다. 
- 객체는 어떤 매개변수도 취하지 않는다
- 잘 어울리는 상황
  - 순수 함수
  - IO 관련 함수
- Apply 메소드와 동반 객체
  - 기본 메소드 혹은 인젝터 메소드로 불리며, 메소드 이름 없이 호출될 수 있음. 
  - ```scala
     class Multiplier(factor:Int) {
       def apply(input: Int) = input * factor
     }
  
     val tripleMe = new Multiplier(3)
     val tripled = tripleMe.apply(10)
     val tripled2 = tripleMe(10)
     ```
    tripled와 tripled2엔 모두 같은 값이 들어간다. 
  - Apply 메소드는 팩토리 패턴으로 자주 쓰인다
    - Ex)
      - List 객체는 인수를 취하고 그로부터 새로운 컬렉션을 반환
      - Option 객체에선 단일 값을 취하여, 그 값이 null이 아닌 값을 포함하면 Some[A], 그렇지 않으면 None을 반환
      - Future 객체에선 매개변수로 함수를 취하여, 해당 함수를 백그라운드 스레드에서 호출
  - 동반객체: 클래스와 동일한 이름을 공유하며, 동일한 파일 내에서 그 클래스로 함께 정의되는 객체(Object)
    - 동반객체와 클래스는 서로의 private, protected 필드와 메소드에 접근 가능하다. 
  - 스칼라에선 클래스의 새로운 인스턴스를 그 동반 객체로부터 생성하는 팩토리 패턴을 사용하는게 일반적
    - 스칼라에선 자바의 static과 같은 키워드가 없기 때문에 동반 객체를 사용. 결국 동반 객체는 클래스의 static 필드와 메소드를 분리해둔 곳
  - ```scala
    class Multiplier(val x:Int) { def product(y:Int) = x * y}
    object Multiplier { def apply(x: Int) = new Multiplier(x) }
    
    val tripler = Multiplier(3)
    ```
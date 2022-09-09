# Implicit

## 묵시적 매개 변수
- 함수 정의시 매개 변수 괄호 외에 묵시적 매개 변수를 위한 괄호 표시 가능(예시의 test 함수 정의 하는 부분에 소괄호 페어가 2개 있음)
- 함수 호출시 묵시적 매개 변수 칸에도 명시적으로 값을 넣을 수 있음
- 함수 호출하는 지역에 implicit으로 선언한 값이나 변수가 있다면 함수 호출 시 따로 묵시적 파라미터 넣지 않아도 자동으로 입력됨.
  - 타입을 기준으로 자동 입력
- 단 implicit 값이나 변수는 동일한 타입으론 2개 이상 생성하면 안됨. 2개 이상 있으면 묵시적 매개변수 사용하는 함수에서 예외 발생. (ambiguous implicit values)
```scala
object Main {
  def main(args: Array[String]): Unit = {
    implicit val b:Int = 2
    implicit val c:Float = 2
    println(test(1))
  }

  def test(a:Int)(implicit b:Int): Int = {
    return a+b
  }
}
```
## 묵시적 클래스
- 특정 클래스에 Implicit 클래스에 정의된 필드나 메소드를 사용할 수 있게 하는 기능
```scala
object Implicit extends App{
  implicit class Fishies(val x:Int){
    def fishes = "Fish" * 3
  }

  println(3.fishes)
}
```
- implicit class의 조건
  - 오직 1개의 constructor parameter만 허용하고 해당 parameter의 타입은 묵시적이지 않은 클래스여야 한다 
  - 다른 객체, 클래스 또는 트레이트 안에서 정의되어야 한다. 
  - 묵시적 클래스명은 현재 네임스페이스의 다른 객체, 클래스, 트레이트의 이름과 충돌하면 안된다.

- 장점
  - 유용한 메소드를 기존 클래스에 추가하는 방법
  - 주의 깊게 사용하면 코드를 더 표현력 있게 만듬
- 단점
  - 가독성 훼손 가능
  
- 특정 클래스에 대한 Implicit 클래스 여러개 만드는 것도 가능
# Implicit

- 묵시적 매개 변수
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
- 묵시적 클래스
# 인스턴스화를 막으려거든 private 생성자를 사용하라

<br>

## 인스턴스화가 불필요한 경우
 - 정적 메서드와 정적 필드만을 담은 클래스(java.lang.Math, java.util.Collections 등)

<br>

## private 생성자로 인스턴스화 방지를 안했을 때 문제
 - 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어 인스턴스화 가능
 - 추상 클래스를 만드는 방법은 하위 클래스를 만들어 인스턴스화 할 수 있음

<br>

## private 생성자로 인스턴스화 방지
```java
public class UtilityClass {
  //기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
  private UtilityClass() {
    throw new AssertionError();
  }
}
```
 - private로 클래스 외부에서 접근 불가
 - AssertionError로 클래스 내부에서도 생성자 호출 불가
 - 생성자가 있지만 호출할 수 없으므로 주석으로 알려줌
 - 상속을 불가능하게 하는 효과도 있음

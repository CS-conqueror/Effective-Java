# 멤버 클래스는 되도록 static으로 만들라
> 불필요한 비정적 멤버 클래스는 공간과 시간이 낭비 됨

<br>

## 중첩 클래스

---

> 다른 클래스 안에 정의된 클래스로 자신을 감싼 외부 클래스에서만 사용

<br>

### 멤버 클래스
 - 외부 클래스의 private 멤버에도 접근 가능
 - 다른 정적 멤버와 똑같은 접근 규칙(private, public 등) 적용

<br>

#### 정적 멤버 클래스
> 외부 클래스와 관련 있지만 인스턴스 멤버에 접근할 필요가 없을 때 사용
```java
class OuterClass {
    private static String staticMember = "Static Member";

    // 정적 멤버에는 접근 가능하지만 인스턴스 멤버에는 접근 불가
    static class StaticNestedClass {
        void display() {
            System.out.println("Outer class static member: " + staticMember);
        }
    }

    public static void main(String[] args) {
        OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();
        nestedObject.display();
    }
}
```

 - 외부 클래스 객체의 구성 요소를 나타낼 때 사용(Map.Entry)

<br>

#### 비정적 멤버 클래스
> 외부 클래스와 관련 있으며 인스턴스 멤버에 접근할 필요가 있을 때 사용, 외부 클래스 인스턴스에 종속
```java
public class OuterClass {
    private String instanceField = "Instance Field";

    // 정적 멤버와 인스턴스 멤버 모두 접근 가능
    public class InnerClass {
        private String innerInstanceMember = "Inner Instance Member";
        public void display() {
            // this를 사용하여 내부 클래스의 멤버에 접근
            System.out.println("Inner class instance member: " + this.innerInstanceMember);

            // 클래스명.this를 사용하여 외부 클래스의 멤버에 접근
            System.out.println("Outer class instance member: " + OuterClass.this.outerInstanceMember);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        // 외부 클래스 인스턴스 메서드로 비정적 멤버 클래스 생성자를 호출하여 만들거나
        // 직접 new를 호출해서 생성
        OuterClass.InnerClass inner = outer.new InnerClass();
        inner.display();
    }
}
```
 - 외부 클래스 인스턴스 없이 생성 불가
 - 외부 클래스 인스턴스로의 숨은 외부 참조를 가져 메모리 누수가 생길 수 있음
 - 공개 클래스(public, private)의 멤버는 비정적 내부 클래스를 차후에 정적 내부 클래스로 변경하기 어려움

<br>

## 익명 클래스
> 이름이 없는 일회성 클래스

```java
interface Greeting {
    void greet();
}

public class Main {
    public static void main(String[] args) {
        // 일회용으로 사용할 인터페이스나 추상클래스를 구현할 때 사용
        // 가독성 주의
        Greeting greeting = new Greeting() {
            @Override
            public void greet() {
                System.out.println("Hello, World!");
            }
        };
        greeting.greet();
    }
}
```
 - 클래스 정의와 생성이 동시에 발생
 - instanceof 검사나 클래스의 이름이 필요한 작업은 수행 불가
 - 여러 인터페이스 구현 및 구현과 동시에 상속 불가
 - 클라이언트는 익명 클래스의 상위 타입에서 상속한 멤버 외에 호출 불가
 - 비정적 문맥에서 사용될 때만 외부 클래스 인스턴스 참조 가능
 - 정적 문맥에서는 상수 변수 이외의 정적 멤버는 가질 수 없음
 - 람다를 활용하여 익명 클래스 대체 가능

<br>

## 지역 클래스
> 메서드 내부에 정의된 클래스로 지역변수와 선언 위치 및 유효 범위가 동일
```java
public class OuterClass {
    public void outerMethod() {
        final String localVariable = "Local Variable";

        class LocalClass {
            public void display() {
                System.out.println("Accessing local variable: " + localVariable);
            }
        }

        LocalClass local = new LocalClass();
        local.display();
    }
}

public class Main {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.outerMethod();
    }
}
```

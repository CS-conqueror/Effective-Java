# 예외는 진짜 예외 상황에만 사용하라

<br>

> 예외를 제어 흐름용으로 사용하면 코드를 헷갈리게 하고 성능을 떨어뜨리고 제대로 동작하지 않을 수 있으므로 예외 상황에만 사용하자

<br>

## 예외를 제어 흐름용으로 사용할 때의 단점

---

 - 코드를 이해하기 어려움
 - 제대로 동작하지 않음
 - 유지보수가 어려움
 - 성능을 떨어뜨리거나 성능이 좋아지더라도 자바 플랫폼의 최적화로 성능 우위는 오래가지 않음

<br>

```java
// 예외로 제어 흐름용으로 사용
try {
  int i = 0;
  while(true)
    range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}

// 표준적인 관용구
for (Mountain m : range)
  m.climb();
```

```java
    public static int findValue(int[] array, int value) {
        // index가 벗어난 경우와 값을 찾지 못한 경우를 구분하지 못함
        for (int i = 0; i <= array.length; i++) {
            if (array[i] == value) {
                return i;
            }
        }
        throw new ArrayIndexOutOfBoundsException("Value not found");
    }
```

<br>

## API에서 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 하는 방법

---

> 상태 의존적 메서드를 제공할 때는 상태 검사 메서드를 제공하거나 빈 옵셔널 값, 특정 값(null 등)을 반환해야 함

```java
public class SafeDivide {
    private int denominator;

    public SafeDivide(int denominator) {
        this.denominator = denominator;
    }

    public boolean canDivide() {
        return denominator != 0;
    }

    // 상태 검사 메서드 제공
    public int divide(int numerator) {
        if (canDivide()) {
            return numerator / denominator;
        } else {
            throw new ArithmeticException("Cannot divide by zero");
        }
    }

    // 빈 옵셔널 반환 
    public Optional<Integer> divide(int numerator) {
        if (denominator == 0) {
            return Optional.empty();
        } else {
            return Optional.of(numerator / denominator);
        }
    }

    // 특정값 반환
    public int divide(int numerator) {
        if (denominator == 0) {
            return Integer.MIN_VALUE;
        } else {
            return numerator / denominator;
        }
    }
}
```

<br>

 ### 선택 가이드

 <br>

 - 상태 검사 메서드와 상태 의존적 메서드 호출 사이에 객체의 상태가 변할 수 있는 경우(외부 동기화 없이 여러 스레드가 동시에 접근, 외부 요인으로 상태가 변할 수 있음) 옵셔널이나 특정값을 사용
 - 상태 검사 메서드와 상태 의존적 메서드가 중복 작업을 수행하고 성능이 중요하다면 옵셔널이나 특정 값을 선택
 - 대부분의 경우 상태 검사 메서드가 가독성이 좋고 유지 보수에 용이

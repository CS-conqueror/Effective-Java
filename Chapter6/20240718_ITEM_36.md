# 비트 필드 대신 EnumSet을 사용하라

> 비트 필드보다 가독성도 좋고 유연하며 여러 기능을 제공하는 EnumSet을 사용하자

<br>

## 비트 필드

---

> 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴

<br>

```java
public class Text {
  public static final int STYLE_BOLD = 1 << 0;
  public static final int STYLE_ITALIC = 1 << 1;
  public static final int STYLE_UNDERLINE = 1 << 2;
  public static final int STYLE_STRIKETHROUGH = 1 << 3;

  public void applyStyles(int styles) { ... }
}
```

1010 -> 두껍게 + 밑줄

### 단점
 - 출력값 해석이 어려움
 - 모든 원소 순회가 어려움
 - 최대 몇 비트가 필요한지 미리 예측해야 함

<br>

## EnumSet

---

> 열거 타입 상수의 값으로 구성된 집합

<br>

```java
public class Text {
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

  public void applyStyles(Set<Style> styles) { ... }
}

text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

### 장점
 - 타입 안전하고 Set을 구현하여 다른 Set 구현체와 같이 사용 가능
 - 내부가 비트 벡터로 구현되어 비트 벡터와 유사한 성능
 - 대량 작업을 위한 메서드(removeAll, retainAll) 제공
 - 비트를 다룰 때 겪는 오류 방지(중복 비트 설정, 잘못된 비트 위치 사용)
 - 가독성 상승

<br>

### 단점
  - 불변 EnumSet을 바로 만들 수 없음 > unmodifiableSet을 활용 

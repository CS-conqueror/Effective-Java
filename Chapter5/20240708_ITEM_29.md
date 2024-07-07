# 이왕이면 제네릭 타입으로 만들라

> 제네릭 타입을 사용하면 형변환을 할 필요가 없어 더 안전하고 편하게 사용 가능

<br>

## 제네릭 타입이 아닌 경우

---

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

스택에서 꺼낸 객체를 사용하기 위해 형변환을 해야 하고 형변환을 적절히 하지 못하면 런타임 오류가 발생

<br>

## 제네릭 타입

---

### 제네릭 클래스 만드는 순서
 1. 클래스 선언에 타입 매개변수를 추가
 2. 코드에 쓰인 Object를 적절한 타입 매개변수로 변경

<br>

 - 타입 매개변수는 주로 E 사용

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        // 오류 발생 : E와 같은 실체화 불가 타입으로는 배열을 만들 수 없음
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

<br>

### 오류 해결 방법
 1. Object 배열을 생성 후 제네릭 배열로 형변환 > 비검사 형변환이 안전한지 확인 후 @SuppressWarnings 애너테이션으로 경고를 숨김
    - 가독성이 좋음
    - 배열 생성 시 한번만 형변환이 일어남
    - 힙 오염을 일으킬 수 있음
```java
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
```
 2. elements 필드의 타입을 E[]에서 Object[]로 변경 > 비검사 형변환이 안전한지 확인 후 @SuppressWarnings 애너테이션으로 경고를 숨김
    - 배열에서 원소를 읽을 때마다 형변환
    - 힙 오염에 안전
```java
private Object[] elements;

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        @SuppressWarnings("unchecked") E result = (E) elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
```

<br>

## 리스트보다 배열을 사용해야 하는 경우

 - 자바에서 리스트는 기본 타입이 아니므로 결국 제네릭 타입은 기본 타입인 배열을 사용해 구현
 - 성능이 우선시된다면 배열을 사용

<br>

## 제네릭 타입의 타입 매개변수
 - 기본 타입(int, double)등은 사용하지 못하므로 박싱된 기본 타입(Integer, Double)을 사용
 - `한정적 타입 매개변수` : E extends Class와 같이 타입 매개변수를 특정 클래스의 하위 타입만으로 제한할 수 있음
 - 한정적 타입 매개변수를 사용하면 원소 사용 시 형변환 없이 바로 클래스의 메서드 사용 가능

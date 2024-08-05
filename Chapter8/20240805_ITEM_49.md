# 매개변수가 유효한지 검사하라

<br>

> 메서드는 최대한 제약 없이 범용적으로 설계해야 하지만 제약이 있다면 가능한 빨리 유효성을 검사하라

<br>

## 1. 매개변수의 제약은 메서드 몸체가 시작되기 전에 검사

---

그렇지 않다면...

 - 매서드가 수행되는 중간에 예외를 던지면 정확한 발생 지점을 찾기 어려움
 - 메서드는 잘 수행되지만 잘못된 결과를 던질 수 있음
 - 메서드로 인해 잘못 변경된 객체가 메서드와 관련 없는 오류를 발생

<br>

### 예외
 - 유효성 검사 비용이 높음
 - 계산 과정에서 암묵적으로 검사가 수행
   * Collections.sort(List)에서 연산 도중 비교가 불가하면 ClassCastException을 던지므로 사전에 모든 객체가 비교 가능한지 검사할 필요 없음

```java
// 유효성 검사 비용이 높음
public class ComplexOperation {
    private int[] data;

    public ComplexOperation(int[] data) {
        // 데이터가 null인지와 길이만 초기 검사를 하고, 나머지 복잡한 검사는 실제 연산 중에 수행
        this.data = Objects.requireNonNull(data, "Data array cannot be null");
        if (data.length == 0) {
            throw new IllegalArgumentException("Data array cannot be empty");
        }
    }

    public int compute(int index) {
        // 복잡한 계산 과정 중에 유효성 검사
        int result = 0;
        for (int i = 0; i < data.length; i++) {
            if (data[i] < 0) {
                throw new IllegalArgumentException("Data array contains negative values");
            }
            result += data[i] * index;
        }
        return result;
    }
}

```

단, 실패 원자성(호출한 메서드가 실패하더라도 객체는 그 전의 상태를 유지해야 함)

```java
// 실패 원자성 보장
    public void transfer(BankAccount target, double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Transfer amount must be positive");
        }
        if (this.balance < amount) {
            throw new IllegalArgumentException("Insufficient balance for transfer");
        }

        this.balance -= amount;
        target.balance += amount;
    }
```

<br>

## 2. 공개 API의 매개변수의 제약은 문서화

 - 매개변수의 제약 뿐 아니라 어겼을 때의 예외를 @throws 자바독 태그를 활용하 문서화
 - 잘못된 매개변수를 사용해서 발생한 예외와 API 문서에서 던지기로 한 예외가 다르다면 예외 번역 관용구를 사용 API 문서에 기재된 예외로 번역

```java
    /**
     * @param initialBalance는 음수가 될 수 없다
     * @throws IllegalArgumentException initialBalance가 음수라면 발생
     */
    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }
```

<br>

## 3. 메서드를 활용하여 매개변수 제약을 자동화
 - requireNonNull : null 검사
 - checkFromIndexSize, checkFromToIndex, checkIndex : 인덱스 검사
 - 비공개 API의 메서드는 단언문(assert)를 사용해 매개변수 유효성 검증
    * 실패하면 AssertionError를 던짐
    * 런타임에 성능 저하가 없음

```java
//requireNonNull 예시, NullPointerException 발생
public class RequireNonNullExample {
    public static String processString(String input) {
        Objects.requireNonNull(input, "Input string cannot be null");
        return input.toUpperCase();
    }
}

//IndexCheck 예시, IndexOutOfBoundsException 발생
public class IndexCheckExample {
// 지정된 시작 인덱스와 길이가 배열 또는 리스트의 범위 내에 있는가
    public static void checkFromIndexSizeExample() {
        int[] array = {1, 2, 3, 4, 5};
        int fromIndex = 2;
        int size = 3;
        Objects.checkFromIndexSize(fromIndex, size, array.length);
    }

// 지정된 시작 인덱스와 종료 인덱스가 배열 또는 리스트의 범위 내에 있는가
    public static void checkFromToIndexExample() {
        int[] array = {1, 2, 3, 4, 5};
        int fromIndex = 1;
        int toIndex = 4;
        Objects.checkFromToIndex(fromIndex, toIndex, array.length);
    }

// 지정된 인덱스가 배열 또는 리스트의 범위 내에 있는가
    public static void checkIndexExample() {
        int[] array = {1, 2, 3, 4, 5};
        int index = 3;
        Objects.checkIndex(index, array.length);
    }
}

// assert 사용
public class PrivateApiExample {
    private void privateMethod(int value) {
        assert value > 0 : "Value must be greater than 0";
        System.out.println("Value is valid: " + value);
    }
}


```

<br>

## 4. 나중에 쓰려고 저장하는 매개변수의 유효성을 검증
 - 나중에는 매개변수가 어디서 왔는지 추적하기 어렵기에 디버깅이 힘듬
 - 생성자가 대표적인 예, 클래스 불변식을 어기지 않기 위해 유효성을 검사

```java
    public Person(String name, String email) {
        this.name = Objects.requireNonNull(name, "Name cannot be null");
        this.email = Objects.requireNonNull(email, "Email cannot be null");

        if (!email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
    }
```

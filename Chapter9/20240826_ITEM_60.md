# 정확한 답이 필요하다면 float와 double은 피하라

<br>

> float와 double은 이진 부동소수점 연산을 근사치로 계산되도록 설계되었으므로 정확한 계산이 필요할 때는 BigDecimal, int, long을 사용하라

<br>

## float와 double의 부적절한 계산 예시

---

```java
    public static void main(String[] args) {
        // 출력값 : 0.09999999999999998
        System.out.println(1.00 - 9 * 0.10);
    }
```

<br>

 위 코드는 1달러로 10센트 사탕 9개를 구매한 경우를 가정한 것인데 출력에서 알 수 있듯이 10센트 사탕 1개를 더 구매할 수 있음에도 계산 결과 구매할 수 없다는 잘못된 결과에 도달

 <br>

 ## 정확한 계산에 사용할 수 있는 타입

---

### BigDecimal

<br>

```java
    public static void main(String[] args) {
        BigDecimal dollar = new BigDecimal("1.00");
        BigDecimal cent = new BigDecimal("0.10");
        // 출력값 : 0.10
        System.out.println(dollar.subtract(cent.multiply(new BigDecimal("9"))));
    }
```
<br>

#### 단점
 - 기본 타입보다 쓰기 불편
 - 속도가 매우 느림

<br>

### int, long 타입

<br>

```java
 public static void main(String[] args) {
        // 1.00의 정수부와 소수부
        int integerPart1 = 1;
        int fractionalPart1 = 0;  // 0.00

        // 0.10의 정수부와 소수부
        int integerPart2 = 0;
        int fractionalPart2 = 10;  // 0.10

        // 9 * 0.10 계산 (정수부와 소수부를 따로 관리)
        int integerPartResult = integerPart2 * 9;
        int fractionalPartResult = fractionalPart2 * 9;

        // 소수부가 100 이상이면 정수부에 반영
        integerPartResult += fractionalPartResult / 100;
        fractionalPartResult = fractionalPartResult % 100;

        // 1.00에서 9 * 0.10을 뺀다 (정수부와 소수부 계산)
        int finalIntegerPart = integerPart1 - integerPartResult;
        int finalFractionalPart = fractionalPart1 - fractionalPartResult;

        // 소수부가 음수가 되면 정수부에서 1을 빌려옴
        if (finalFractionalPart < 0) {
            finalIntegerPart -= 1;
            finalFractionalPart += 100;
        }

         // 출력값 : 0.10
        System.out.println(finalIntegerPart + "." + String.format("%02d", finalFractionalPart));
    }
```

위와 같이 정수부와 소수부를 나눠서 관리할 수도 있고 위의 예시는 단위를 달러 대신 센트로 계산하여 100 - 9 * 10으로 간단하게 정수 타입으로 계산이 가능

<br>

#### 단점
 - 다룰 수 있는 값의 크기가 제한
   * 아홉자리 십진수는 int, 열여덟자리 십진수는 long, 그 이상은 BigDecimal을 사용
 - 소수점을 직접 관리

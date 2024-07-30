# 스트림은 주의해서 사용하라

<br>

> 스트림과 반복을 둘 다 사용해보고 더 나은 쪽을 사용하라

<br>

## 스트림 파이프라인

---

`스트림` : 데이터의 연속적인 흐름
 - 스트림의 원소는 다양한 데이터 소스(컬렉션, 배열, 파일 등)에서 오는 객체 참조나 기본 타입(int, long, double)

`스트림 파이프라인` : 스트림의 원소로 수행하는 연산 단계

<br>

### 스트림 파이프라인 특징

<br>

- 연산을 하여 새로운 스트림을 만드는 여러 중간 연산과 스트림을 소모하여 최종 결과를 생성하는 종단 연산으로 이루어짐
- 종단 연산에 사용되는 데이터 원소만 계산에 사용(`지연평가`)
- 파이프라인 하나를 구성하는 모든 호출을 연결하여 하나의 표현식으로 표현 가능
- 기본적으로는 순차적으로 수행, parallel 메서드로 병렬 수행도 가능
- chars()가 반환하는 스트림 원소는 char가 아닌 int이므로 char값 처리 시 스트림 사용 지양 권장

<br>

## 사용 예시

---

### 스트림을 사용하면 좋은 경우

<br>

- 중간 연산(필터링, 매핑, 정렬, 제한 등)
- 최종 연산(수집, 연산, 검색 등)

```java
public class StreamExample {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("stream", "filter", "map", "flatMap", "distinct", "sorted", "limit", "skip");

        List<String> result = words.stream()
            .filter(word -> word.length() > 4)     // 중간 연산: 필터링
            .distinct()                            // 중간 연산: 필터링
            .map(String::toUpperCase)              // 중간 연산: 매핑
            .flatMap(word -> Arrays.stream(word.split(""))) // 중간 연산: 평탄화
            .sorted()                              // 중간 연산: 정렬
            .limit(20)                             // 중간 연산: 제한
            .skip(2)                               // 중간 연산: 제한
            .forEach(System.out::println)          // 종단 연산
    }

    // 이 외에도 종단 연산은 collect(수집), sum(연산), findFirst(검색) 등이 존재
}
```

<br>

- 위의 예에서 flatMap을 사용한 부분에 map을 사용했으면 `Stream<Stream<String>`이 반환되고 flatMap을 사용하면 `Stream<String>`이 반환

<br>

### 스트림을 사용하기 안좋은 경우

<br>

 - 함수 객체(람다, 메서드 참조)로 표현할 수 없는 경우
   * 지역 변수를 수정하는 경우(final이거나 사실상 final인 변수만 읽을 수 있음)
   * return, break, continue의 사용이 필요한 경우
   * 메서드에 명시된 검사 예외를 던져야 하는 경우
 - 스트림을 사용함으로 오히려 코드의 가독성이 떨어지는 경우
   * 람다에서 매개변수 타입을 생략하므로 명확한 이름을 사용
   * 도우미 메서드를 활용
 - 한 데이터를 파이프라인 여러 단계에서 접근이 필요할 때 -> 앞 단계의 값이 필요하다면 거꾸로 매핑을 수행

<br>

#### 가독성이 떨어지는 경우
```java
 try (Stream<String> words = Files.lines(dictionary)) {
       words.collect(
              // 가독성이 떨어짐
               groupingBy(word -> word.chars().sorted()
                       .collect(StringBuilder::new,
                               (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
               .values().stream()
               .filter(group -> group.size() >= minGroupSize)
               .map(group -> group.size() + ": " + group)
               .forEach(System.out::println);
        }
    }
}
```

<br>

```java
 try (Stream<String> words = Files.lines(dictionary)) {
      // 도우미 메서드 활용
       words.collect(groupingBy(word -> alphabetize(word)))
               .values().stream()
               .filter(group -> group.size() >= minGroupSize)
               .forEach(g -> System.out.println(g.size() + ": " + g));
  }

    public static String alphabetie(String s) {
    	char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```

<br>

#### 한 데이터가 여러 파이프라인에서 필요한 경우
```java
public void mersenne() {
    primes()
            .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(10)
            // 앞 단계의 값을 구하기 위해 거꾸로 매핑
            .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
}
```

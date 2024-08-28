# 객체는 인터페이스를 사용해 참조하라

<br>

> 유연하고 유지보수에 용이한 프로그램을 위해 적합한 인터페이스가 있다면 객체는 인터페이스를 이용해 참조하자

<br>

## 인터페이스 참조 이유

---

> 구현 클래스 교체 시 생성자 부분만 수정하면 됨

- 선언 타입도 변경 시 기존에 제공하던 메서드를 새 클래스 타입이 제공하지 않으면 컴파일 에러 발생
- 구현 클래스 교체시에는 인터페이스 일반 규약 이외의 특별한 기능을 제공한다면 새로운 클래스도 같은 기능을 제공해야 함

<br>

- 타입이 인터페이스인 경우
```java
    public static void main(String[] args) {
        //LinkedList로 변경 시 new ArrayList<>() 부분만 new LinkedList<>()로 변경하면 됨
        List<String> items = new ArrayList<>();
        items.add("Item1");
        items.add("Item2");
        System.out.println(items.get(0));
    }
```
- 타입이 클래스인 경우
```java
    public static void main(String[] args) {
        //LinkedList로 변경 시 ArrayList가 쓰인 모든 부분을 LinkedList로 변경
        //ArrayList에서 쓰이는 trimToSize를 사용했기에 LinkedList로 변경 후 컴파일 에러 발생
        ArrayList<String> items = new ArrayList<>();
        items.add("Item1");
        items.add("Item2");
        items.trimToSize();
        System.out.println(items.get(0));
    }
```

<br>

## 클래스를 참조하는 경우

---

 - 값 클래스 : String, Builder
 - 클래스 기반으로 작성된 프레임워크가 제공하는 객체 : OutputStream, HttpServlet, Thread
 - 인터페이스에 없는 특별한 메서드를 제공하는 클래스 : PriorityQueue, TreeSet

클래스를 참조하는 경우에도 계층구조 중 필요한 기능을 만족하는 가장 추상적인 클래스 타입을 사용

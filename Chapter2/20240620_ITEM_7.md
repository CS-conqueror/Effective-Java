# 다 쓴 객체 참조를 해제하라

<br>

> 자바에서 가비지 컬렉터가 메모리를 자동으로 관리해주지만 메모리 관리에 전혀 신경 쓰지 않아도 되는 것은 아님

강한 참조를 하고 있는 상태라면 가비지 컬렉터는 메모리에서 자동으로 객체를 제거하지 않음

<br>

## 메모리 누수 원인
 ### 1. 다 쓴 객체의 참조
```java
public class IntegerList {
    private int[] numbers;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 10;
    
    IntegerList() {
        numbers = new int[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(int number) {
        // 원소 공간 확보 로직 생략
        numbers[size++] = number;
    }
    
    public int removeLast() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return numbers[--size];
        // 사용하지 않는 숫자들도 여전히 참조를 하고 있어 메모리 누수 발생 가능
    }  
}
```

<br>

---
#### 해결 방안

<br>

 ##### 참조를 담은 변수를 유효 범위 밖으로 밀어 냄

유효 범위를 너무 넓게 잡아서 반복 후에도 iterateSize 변수가 메모리에 남음
 ```java
public class Main {
    private static int iterateSize = 3;
    public static void main(String[] args) {
        List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        for (int index = 0; index < iterateSize; index++) {
         System.out.println(list.get(index));
        }
    }
}
```

<br>

유효 범위를 좁게 하여 반복 후에 iterateSize 변수가 제거되도록 함
```java
public class Main {
    public static void main(String[] args) {
        int iterateSize = 3;
        List<Integer> list = List.of(1, 2, 3, 4, 5, 6, 7);
        for (int index = 0; index < iterateSize; index++) {
         System.out.println(list.get(index));
        }
    }
}
```


<br>

 ##### 참조를 다 썼을 때 null 처리

 ```java
public class IntegerList {
    private int[] numbers;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 10;
    
    IntegerList() {
        numbers = new int[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(int number) {
        // 원소 공간 확보 로직 생략
        numbers[size++] = number;
    }
    
    public int removeLast() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        numbers[size] = null; // 다 쓴 참조 해제제
        return numbers[--size];
    }  
}
```

<br>

### 2. 캐시
```java
import java.util.HashMap;
import java.util.Map;

public class CacheMemoryLeakExample {
    private Map<String, Object> cache = new HashMap<>();

    public void addToCache(String key, Object value) {
        cache.put(key, value);
    }

    public Object getFromCache(String key) {
        return cache.get(key);
    }

    public static void main(String[] args) {
        CacheMemoryLeakExample example = new CacheMemoryLeakExample();
        
        for (int i = 0; i < 10000; i++) {
            example.addToCache("key" + i, new Object());
        }
        
        System.out.println("Cache size: " + example.cache.size());
        // 메모리 누수 발생: 캐시가 모든 객체의 강한 참조를 유지하고 있음
    }
}
```
<br>

#### 해결 방안
 - WeakHashMap을 사용하여 더 이상 강한 참조를 하지 않는 객체를 가비지 컬렉터가 자동으로 제거
 - 백그라운드 스레드를 활용하여 주기적으로 캐시를 청소
 - 새 엔트리를 추가할 때 부수 작업 수행
   ex) 캐시가 일정 크기를 초과하면 오래된 항목을 제거

<br>

### 3. 리스너, 콜백
```java
import java.util.ArrayList;
import java.util.List;

interface EventListener {
    void onEvent();
}

class EventSource {
    private List<EventListener> listeners = new ArrayList<>();

    public void registerListener(EventListener listener) {
        listeners.add(listener);
    }
}

public class MemoryLeakExample {
    public static void main(String[] args) {
        EventSource eventSource = new EventSource();
        EventListener listener = new EventListener()
        eventSource.registerListener(listener);

        // listener에 대한 참조가 계속 유지되어 메모리 누수 발생 가능
        listener = null;

        // 이 시점에서 메모리 누수가 발생할 수 있음
        System.out.println("Listener registered, but not unregistered.");
    }
}
```

<br>

#### 해결 방안 : 약한 참조로 저장하여 가비지 컬렉터가 자동으로 제거
```java
import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

interface EventListener {
    void onEvent();
}

class EventSource {
    private List<WeakReference<EventListener>> listeners = new ArrayList<>();

    public void registerListener(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }
}

public class MemoryLeakSolution {
    public static void main(String[] args) {
        EventSource eventSource = new EventSource();
        EventListener listener = new EventListener()
        eventSource.registerListener(listener);

        // listener에 대한 참조를 제거
        listener = null;

        System.out.println("Listener registered and eligible for GC.");
    }
}
```

 



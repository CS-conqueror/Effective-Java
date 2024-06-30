# 상속보다는 컴포지션을 사용하라
 - 여기에서 상속은 구현 상속(extends)를 의미
<br>

## 상속의 문제점

---

 - 상위 클래스 구현에 따라 하위 클래스 동작에 이상이 생길 수 있음
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```
 * addAll 실행 시 3개를 더해도 addAll은 6이 증가(HashSet이 add를 사용하기 때문)
 * 이에 맞게 수정을 하더라도 이후에 HashSet의 addAll 방식이 변경되면 문제 발생 가능
 * 하위 클래스 재정의
   - addAll을 하위클래스에서 새로 구현하는 것은 오류를 낼 수 있고 상위 클래스의 private 필드는 사용이 불가
   - 상위 클래스에서 차후에 추가 조건이 생겼을 때 하위 클래스에서 기존에 재구현한 메서드는 그 조건을 적용하지 못 함
 * 새로운 매서드 추가
   - 차후에 상위 클래스에 메서드와 시그니처가 같은 메서드가 추가되면 컴파일 되지 않거나 재정의 한 것과 같음
   - 상위 클래스에서 요구하는 규약을 만족 못할 가능성 높음

<br>

## 컴포지션

---

 > 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조

- 전달 : 새 클래스에 인스턴스 메서드들은 기존 클래스의 메서드를 호출

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {this.s = s;}

    @Override
    public boolean add(E e) {retunr s.add(e);}

    @Override
    public boolean addAll(Collection<? extends E> c) {retunr s.addAll(c);}
}

import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

public class InstrumentedHashSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}
```
- 이미 Set 인터페이스를 구현한 어떤 구현체를 이용하여 그 기능을 안전하게 사용 가능
- 래퍼 클래스는 콜백 프레임워크에 사용하면 SELF 문제가 발생하는 단점 존재
```java
interface SomethingWithCallback {
      void doSomething();
      void call();
    }

    class WrappedObject implements SomethingWithCallback {

      private final SomeService service;

      WrappedObject(SomeService service) {
        this.service = service;
      }

      @Override
      public void doSomething() {
        service.performAsync(this);
      }

      @Override
      public void call() {
        System.out.println("WrappedObject callback!");
      }
    }


    class Wrapper implements SomethingWithCallback {

      private final WrappedObject wrappedObject;

      Wrapper(WrappedObject wrappedObject) {
        this.wrappedObject = wrappedObject;
      }

      @Override
      public void doSomething() {
        wrappedObject.doSomething();
      }

      void doSomethingElse() {
        System.out.println("We can do everything the wrapped object can, and more!");
      }

      @Override
      public void call() {
        System.out.println("Wrapper callback!");
      }
    }

    final class SomeService {

      void performAsync(SomethingWithCallback callback) {
        new Thread(() -> {
          perform();
          callback.call();
        }).start();
      }

      void perform() {
        System.out.println("Service is being performed.");
      }
    }
    public static void main(String[] args) {
        SomeService   service       = new SomeService();
        WrappedObject wrappedObject = new WrappedObject(service);
        Wrapper       wrapper       = new Wrapper(wrappedObject);
        // Wrapper의 doSomething을 호출해도 WrappedObject doSomething이 실행
        wrapper.doSomething();
    }   
```

<br>

## 상속 사용 주의 사항
 - 하위 클래스가 진짜 상위 클래스의 하위 타입인가?(is-a 관계)
 - 확장하려는 클래스의 API에 결함이 없는가
 - 결함이 있다면 다른 클래스의 API에 전파돼도 괜찮은가?

  ### 잘못 사용한 예시
  - Vector와 Stack : add메서드를 통해 원소를 마지막이 아닌 위치에 삽입할 수 있음음
  - Hashtable과 Properties : get메서드를 통해 문자열 이외 키와 값도 사용 가능하고 불변식도 깰 수 있음

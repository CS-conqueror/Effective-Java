# null이 아닌, 빈 컬렉션이나 배열을 반환하라

<br>

> null을 반환하는 API는 오류 처리가 필요하지만 성능이 그렇게 좋아지지도 않음

<br>

## 1. null 반환의 단점

---

 - 방어 코드를 항상 넣어줘야 함
 - 방어 코드가 없다면 NullPointException가 발생할 수 있음

<br>

```java
private final List<User> users;

public List<User> getUsers() {
  return users.isEmpty() ? null : new ArrayList<>(users);
}

public void printUsers() {
  List<User> users = getUsers();
  if (users == null) System.out.println("유저가 없어요!);
  else // users 출력 코드 생략
}
```

<br>

## 2. 빈 컨테이너를 할당해도 되는 이유

---

 - 대부분의 경우 성능 차이가 거의 없음
 - 성능 차이가 있는 경우 똑같은 빈 컨테이너를 반환하면서 해결 가능

<br>

```java
private final List<User> users;
private static final User[] EMPTY_USER_ARRAY = new User[0];

public List<User> getUsers() {
  return users.isEmpty() ? Collections.emptyList() : new ArrayList<>(users);
}

// toArray 인자의 크기보다 더 큰 배열은 새로 생성해서 반환하고 그 이외는 입력한 배열에 원소를 담아서 반환
public User[] getUserArray() {
  return users.toArray(EMPTY_USER_ARRAY);
}

```

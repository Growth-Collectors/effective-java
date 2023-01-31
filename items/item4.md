# [Item 4] 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담은 클래스의 경우, (ex) `java.lang.Math`, `java.util.Arrays` 유틸리티 클래스) 인스턴스로 만들어서 사용하려고 설계한 클래스가 아니다
- 그런데, 생성자를 명시하지 않으면 자바 컴파일러가 자동으로 **기본 생성자**를 만들어준다
   - 자동으로 기본 생성자 (default constructor)가 만들어지는 경우에, 사용자는 의도치않게 유틸리티 클래스를 인스턴스화 할 수 있다

```java
public class UtilityClass {

  public static String today() {
    return "2023-01-01";
  }

  public static void main(String[] args) {
    UtilityClass uc = new UtilityClass(); // 아예 인스턴스를 못만들게 해보자
    uc.today();
  }
}
```

### 추상 클래스로 만드는 것은 어떨까?
```java
public abstract class UtilityClass {
  public UtilityClass () {
    System.out.println("default constructor");
  }

  public static String today() {
    return "2023-01-01";
  }

  public static void main(String[] args) {
    String today = UtilityClass.today();
  }
}
```

- 추상 클래스의 서브 클래스를 만들어서 인스턴스화 하면 막을 수 없다
```java
public class SubClass extends UtilityClass {
  public static void main(Stirng[] args) {
    SubClass sc = new SubClass();
  }
}
```
- 자식 클래스의 인스턴스를 만들 때, 부모 클래스의 기본 생성자를 호출하게 되므로, 인스턴스화를 막을 수가 없다
- 오히려, `abstract` 라는 키워드로 인해, 상속해서 쓰라는 뜻으로 오해할 수도 있다

### 📌 결론
- `private` 생성자를 추가해서, 클래스의 인스턴스화를 막으면 된다
```java
public class UtilityClass {

  // 기본 생성자가 만들어지는 것을 막는다 (인스턴스화 방지용)
  private UtilityClass() {
    throw new AssertionError();
  }

  public static String today() {
    return "2023-01-01";
  }

  public static void main(String[] args) {
    String today = UtilityClass.today();
    // UtilityClass uc = new UtilityClass();
    // uc.today();
  }
}
```
- 명시적 생성자가 private 이어서 클래스 바깥에서는 접근할 수 없다
- AssertionError를 통해서 클래스 내부에서 실수로 생성자를 호출하지 않도록 한다
- 또한, 이렇게 하면, **상속을 불가능**하게 하는 효과도 생긴다
  - 왜?
    - 모든 생성자는 명시적이든 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, 이를 `private` 으로 선언해버려서 하위 클래스에서 상위 클래스의 생성자를 호출할 수가 없게 된다

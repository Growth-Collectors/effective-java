# [Item 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

```java
public class SpellChecker {
  private static final Dictionary dictionary = new Dictionary();

  private SpellChecker() {} // 객체 생성 방지

  public static boolean isValid(String word) {
    return dictionary.contains(word);
  }
  public static List<String> suggestions(String typo) {
    return dictionary.closeWordTo(typo);
  }
}
```
- 사전을 한국어 사전에서 영어 사전으로 바꾸는 경우?
- 자원을 직접 명시하므로 (자원을 직접 new) 사전을 바꾸는 경우 복잡해진다
  - 테스트 코드를 만들어도, 계속 Dictionary 인스턴스를 생성하고 있으므로...

```java
public class SpellChecker {
  private static final Dictionary dictionary = new Dictionary();

  private SpellChecker() {} // 객체 생성 방지
  public static SpellChecker INSTANCE = new SpellChecker();

  public static boolean isValid(String word) {
    return dictionary.contains(word);
  }
  public static List<String> suggestions(String typo) {
    return dictionary.closeWordTo(typo);
  }
}
```

### 사용하는 자원에 따라 동작이 달라지는 클래스에는 의존 객체 주입을 사용하기
- 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식을 사용하지 말고,
  - 여러 자원 인스턴스를 지원하고, 클라이언트가 원하는 dictionary를 사용해야 하는 경우들..
- 인스턴스를 생성할 때, **생성자에 필요한 자원을 넘겨주는 방식**을 사용하자

```java
public class SpellChecker {
  private final Dictionary dictionary;

  public SpellChecker(Dictionary dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }

  public static boolean isValid(String word) {
    return dictionary.contains(word);
  }
  public static List<String> suggestions(String typo) {
    return dictionary.closeWordTo(typo);
  }
}
```

#### 생성자에 자원 팩터리를 넘겨주는 방식
```java
public SpellChecker(DictionaryFactory dictionaryFactory) {
  this.dictionary = dictionaryFactory.get();
}
```

### 결론
- 사용하는 자원에 따라 동작이 달라지는 클래스에는 필요한 자원을(혹은 자원을 만들어주는 팩터리를) 생성자(혹은 정적 팩터리나 빌더)에 넘겨주도록 한다
- 이러한 의존 객체 주입 기법을 사용하면 클래스의 유연성, 재사용성, 테스트 용이성이 높아진다

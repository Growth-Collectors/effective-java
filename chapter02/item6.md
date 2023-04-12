# [Item 6] 불필요한 객체 생성을 피하라
- 똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나를 재사용하는 것이 나을 때가 많다

### 1. String 인스턴스
```java
String s = new String("bikini");
```
- 이 코드는 실행될 때 마다 String 인스턴스를 새로 만든다
- 생성자에 넘겨진 문자열 자체가 이 생성자로 만들어내려는 String과 기능적으로 완전히 똑같으므로, String 인스턴스를 매번 새로 만드는 것은 불필요한 행위이다
```java
String s1 = new String("bikini");
String s2 = "bikini";
System.out.println(s1 == s2); // false (서로 같은 문자열 리터럴을 가지고 있음에도 불구하고, 서로 다른 인스턴스이다)
```

```java
String s = "bikini";
```
- 이렇게 String 인스턴스를 사용하면, 매번 새로운 String 인스턴스를 만드는 대신에, 하나의 String 인스턴스를 사용한다
- 이 String 인스턴스는 같은 JVM 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용하는 것이 보장된다

---

### 2. 생성자 대신 정적 팩터리 메서드의 사용
- 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해서 불필요한 객체 생성을 피할 수 있다
```java
Boolean.valueOf(String)
```
을 이용하는 것이 좋다!!
- 왜?
  - 생성자는 호출할 때 마다 새로운 객체를 만들지만, 팩터리 메서드는 호출할 때 마다 새로운 객체를 만드는 것이 아니다

### 3. 값 비싼 객체를 캐싱하여 재사용하기
```java
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X|[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
- String.matches 를 사용하는 것은 성능이 중요한 상황에서 반복해서 사용하기에는 적합한 방법이 아니다
- 이 메서드가 내부에서 만드는 정규 표현식용 `Pattern` 인스턴스는, 한 번 쓰고 버려져서, 가비지 컬렉션 대상이 된다
- Pattern은 입력받은 정규 표현식에 해당하는 유한 상태 머신을 만들기에 인스턴스 생성 비용이 높다

##### 불변인 Pattern 인스턴스를 클래스 초기화 (정적 초기화) 과정에서 직접 생성해서 캐싱해두자

```java
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X|[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

  static boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
  }
}
```
- 이렇게, 인스턴스를 **캐싱**해두고, 이 인스턴스를 **재사용**하게 되면 성능을 개선할 수 있다
- 그리고, Pattern 인스턴스를 static final 필드로 선언하게되어, 코드의 의미를 더 잘 드러낼 수 있다
---
### 4. Boxing 된 기본 타입 보다는 기본 타입을 사용하고, 의도치 않은 autoboxing을 주의하기
- 기본 타입과 boxing된 기본 타입을 섞어 쓸 때 자동으로 변환해주는 autoboxing 이 불필요한 객체를 만들어낼 수 있다
```java
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    sum += i;
  }
  return sum;
}
```
- `sum` 변수를 기본 타입이 아닌 박싱된 기본 타입 (`Long`) 으로 선언해서, 불필요한 Long 인스턴스가 2^31 개나 만들어지고 있다

- Boxing된 기본 타입 보다는 기본 타입을 사용하고 의도치 않은 autoboxing을 주의해야 한다

#### 객체 생성이 비싸니까 하지 말아라 ! <- 이 말이 아니다
- 요즘 JVM에서는 객체 생성 및 회수가 큰 부담이 되지는 않는다
- 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것은 좋다
- 그리고, 단순히 객체 생성을 피하고자 개발자 개인적인 객체 pool을 만들지는 말자

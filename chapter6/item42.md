# item 42 익명 클래스 보다는 람다를 사용하라

### 핵심 정리

- 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있다.
- 람다를 사용하지 못하는 경우 및 사용하는 것이 적절하지 않은 경우에만 익명 클래스를 사용하자.
- 익명 클래스는 타입의 인스턴스를 만들 때만 사용하라

자바 8 이전에는 익명 클래스 사용 

```java
@Test
@DisplayName("자바8 이전에 사용하던 익명클래스")
void anonymousClass() {
  List<String> word = new ArrayList<>(List.of("ab","abc","a"));
  
  Collections.sort(word, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
      return Integer.compare(o1.length(), o2.length());
    }
  });
  
}
```

자바 8 이후부터 **인터페이스 내에 추상 메서드가 하나만 존재하면 람다식 사용 가능** 

```java
Collections.sort(word,(s1,s2)->Integer.compare(s1.length(),s2.length()));
```

람다 표현식 사용하면 타입 생략 가능

- 이유 : 컴파일러가 대신 인터페이스에 대한 타입추론 해줌
- 컴파일러가 타입 추론을 할 때 정보를 얻는 방법은 **대부분 제네릭**을 통해 얻음
- List<String>이 아니라 List였으면 컴파일 오류가 발생

```java
@Test
@DisplayName("로 타입 List 임으로 타입추론 불가(컴파일에러)")
void compileError(){
  List word = new ArrayList(List.of("ab","abc","a"));
  Collections.sort(word,(s1, s2) -> Integer.compare(s1.length(),s2.length()));
}
```

해당 코드에서 s1, s2 의 타입 추론 할 수 없음으로 String 의 length()를 호출할 수 없다.

## 열거 타입에서의 람다 사용

상수마다 메서드의 행동을 다르게 해야하는 경우 

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator operator;

    Operation(String symbol, DoubleBinaryOperator operator) {
        this.symbol = symbol;
        this.operator = operator;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return operator.applyAsDouble(x, y);
    }
}
```

- 람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.
- 단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다.
- 그런 다음 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 된다.

### 람다 사용할 때 고려해야 할 점

- 메서드나 클래스와 달리, **람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.**
    - 람다는 한 줄일 때 가장 좋고 길어야 세 줄안에 끝내는 게 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠진다.
- 열거 타입 생성자에 넘겨지는 인수들의 타입도 **컴파일 타임에** 추론된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다(인스턴스는 런타임에 만들어지기 때문이다).
    - 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.
- 람다는 **함수형 인터페이스**에서만 쓰인다.
    - 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다.
    - 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.
- 람다는 자신을 참조할 수 없다.
    - 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
    - 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
    - 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.
- 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있기 때문에, **람다를 직렬화하는 일은 극히 삼가야 한다(익명 클래스의 인스턴스로 마찬가지다).**
    - 만약 직렬화해야만 하는 함수 객체가 있다면(가령 Comparator처럼) private 정적 중첩 클래스의 인스턴스를 사용하도록 한다.

## 핵심 정리

- 자바 8에서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었으며, 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있다.
- **익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하도록 한다.**

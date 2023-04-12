# 아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### 열거 타입과 확장

열거 타입은 책 초판에서 소개한 타입 안전 열거 패턴보다 우수하다.
하지만 타입 안전 열거 패턴은 확장할 수 있지만 열거 타입은 그럴 수 없다.

사실 대부분 상황에서 열거 타입을 확장하는 것은 좋지 않다.
확장한 타입의 원소를 기반 타입의 원소로 취급하지만 그 반대는 성립하지 않는다. 또한 기반 타입과 확장된 원소 모두를 순회할 방법도 마땅치 않다. 그리고 확장성을 높이면 고려할 요소가 늘어나 설계와 구현이 더 복잡해진다.

### 확장할 수 있는 열거 타입

하지만 연산 코드에서는 확장할 수 있는 열거 타입이 어울린다. 기본 아이디어는 열거 타입이 인터페이스를 구현할 수 있다는 사실을 이용하는 것이다. 연산 코드용 인터페이스를 정의하고 열 거 타입이 이 인터페이스를 구현하게 하면 된다.

```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

열거 타입인 BasciOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있다.
연산 타입을 확장해서 지수 연산과 나머지 연산을 추가해보자.
새로 작성한 이 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.
Operation 인터페이스를 사용하도록 작성되어 있기만 하면 된다.

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

개별 인스턴스 수준에서 뿐만 아니라 타입 수준에서도 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용할 수도 있다. main 메서드는 test 메서드에 ExtendedOperation class 리터럴을 넘겨서 확장된 연산들이 무엇인지 알려준다.

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
```

```java
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

test 메서드가 좀 복잡한데 Class객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻이다.
열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행할 수 있기 때문이다.

```java
private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y)
```

두 번째 대안은 Class 객체 대신 한정적 와일드카드 타입인 Collection<? extends Operation>을 넘기는 방법이다. 이 방법은 그나마 간단하다.

```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
}
```

### 인터페이스를 이용해 확장 가능한 열거 타입 흉내 단점

- 열거 타입끼리 구현을 상속할 수 없다.
- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.
- BasicOperation과 ExtendedOperation 모두에 들어가야만 하는 경우 중복량이 적어서 이 경우는 괜찮지만, 공유하는 기능이 많다면 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.
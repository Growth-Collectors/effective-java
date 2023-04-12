# item32. 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 가변 인수(varargs) 메서드
- 제네릭과 함께 자바 5에 추가
- 메서드에 넘기는 인수의 갯수를 클라이언트가 조절할 수 있게 하지만, 구현 방식에 허점이 있다.
- 가변 인수 메서드를 호출하면 가변 인수를 담기 위한 배열이 자동으로 하나 만들어진다.
- 그리고 이 배열이 클라이언트에 노출된다.
    - 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입(실체화 불가 타입)이 포함되면 알기 어려운 컴파일 오류가 발생한다.
    - 거의 모든 제네릭과 매개변수화 타입은 실체화 불가 타입이다.
        - 실체화 불가 타입은 런타임에는 컴파일 타임보다 정보를 적게 담고 있다.

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 
    - 이렇게 다른 타입 객체를 참조하는 상황에서는 **컴파일러가 자동 생성환 형변환이 실패**할 수 있으니, 제네릭 타입 시스템이 약속한 타입 안전성이 무너진다.

- 타입 안전성이 깨지니 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.
    - 그러나 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문에 자바는 이를 허용한다. 컴파일 레벨에서 제어하지 않고 모순을 수용한다. (단순 경고) 

> 예를 들어, 자바 라이브러리는 이런 메서드를 여럿 제공한다. 
>    - Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T...elements), EnumSet.of(E first, E... rest) // 이들은 타입 안전하다.

## @SafeVarags 애너테이션
### 역할
- 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대한 처리 방안이 없다. 
    - 따라서 사용자는 이 경고들을 없애려면 호출하는 곳 모두 SuppressWarnings("unchecked") 애너테이션을 달아야 했다.
- 자바 7부터 @SafeVarargs 애너테이션이 추가되어, 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있다.
- 다만, 메서드가 안전한 게 확실하지 않다면 절대 @SafeVarargs 애너테이션을 달아서는 안 된다. 

### 제네릭 varargs 매개변수와 활용
- 제너릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 단다.
    - 사용자를 헷갈리게 하는 컴파일러 경고를 없앨 수 있다.
    - 안전한 경우가 아니면 제너릭이나 매개변수화 타입의 varargs 매개변수를 사용하지 말아야 한다.
- 재정의할 수 없는 메서드에만 달아야 한다. 
    - 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. 
    - 자바 8에서는 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 private 인스턴스 메서드에도 허용된다.

### 안전하지 않은 가변인수 메서드 예시
```java
public class PickTwo {
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // Can't get here
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("Good", "Fast", "Cheap");
        System.out.println(Arrays.toString(attributes));
    }
}
```
- pickTwo에서는 **Object[]** 를 반환한다. 
    - Object[]는 String[]의 하위 타입이 아니므로 pickTwo의 반환값을 attributes에 저장하기 위해 형변환하다가 ClassCastException을 던진다.
- 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.

### 안전한 경우란?
- **순수하게 인수들을 전달하는 일만 수행하는 경우 안전하다.**
    - varargs 매개변수 배열에 아무것도 저장하지 않는다.
    - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

#### 매개변수 배열을 다른 메서드에서 사용할 수 있는 예외 2가지
- @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
- 배열 내용의 일부 함수 호출만 하는 (varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

```java
// 코드 32-3 제네릭 varargs 매개변수를 안전하게 사용하는 메서드 (195쪽)
public class FlattenWithVarargs {
    @SafeVarargs
    static <T> List<T> flatten(List<? extends T>... lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(
                List.of(1, 2), List.of(3, 4, 5), List.of(6,7));
        System.out.println(flatList);
    }
```

### 제네릭 varargs 매개변수를 List로 대체
- 배열이 없어서 안전하다.
- 정적 팩터리 메서드인 List.of를 활용하면 임의 개수의 인수를 넘길 수 있다. 
    - List.of에도 @SafeVarargs 애너테이션이 달려있다.

```java
// 코드 32-4 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다. (195-196쪽)
public class FlattenWithList {
    static <T> List<T> flatten(List<List<? extends T>> lists) {
        List<T> result = new ArrayList<>();
        for (List<? extends T> list : lists)
            result.addAll(list);
        return result;
    }

    public static void main(String[] args) {
        List<Integer> flatList = flatten(List.of(
                List.of(1, 2), 
                List.of(3, 4, 5), 
                List.of(6,7)));
        System.out.println(flatList);
    }
}

public class SafePickTwo {
    static <T> List<T> pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return List.of(a, b);
            case 1: return List.of(a, c);
            case 2: return List.of(b, c);
        }
        throw new AssertionError();
    }

    public static void main(String[] args) {
        List<String> attributes = pickTwo("Good", "Fast", "Cheap");
        System.out.println(attributes);
    }
}
```
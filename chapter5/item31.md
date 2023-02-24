# item31. 한정적 와일드카드를 사용해 API 유연성을 높이라
    - List<String>은 List<Object> 의 하위 타입이 될 수 없다. 왜냐면 List<Object>가 하는 일을 수행할 수 없기 때문이다. 
- 매개변수화 타입은 **불공변(invariant)** 이다.
    - 리스코프 치환 원칙에 따른다.


## 한정적 와일드 카드 타입

- 불공변 방식보다 유연한 타입이 필요할 때, 활용한다.
- 특별한 매개변수화 타입

```java
// 와일드카드 타입을 이용해 대량 작업을 수행하는 메서드를 포함한 제네릭 스택 (181-183쪽)
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    // 코드 29-3 배열을 사용한 코드를 제네릭으로 만드는 방법 1 (172쪽)
    // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
    // 따라서 타입 안전성을 보장하지만,
    // 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
    @SuppressWarnings("unchecked") 
        public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

//    // 코드 31-1 와일드카드 타입을 사용하지 않은 pushAll 메서드 - 결함이 있다! (181쪽)
//    public void pushAll(Iterable<E> src) {
//        for (E e : src)
//            push(e);
//    }

     // 코드 31-2 E 생산자(producer) 매개변수에 와일드카드 타입 적용 (182쪽)
    public void pushAll(Iterable<? extends E> src) {
        for (E e : src)
            push(e);
    }

//    // 코드 31-3 와일드카드 타입을 사용하지 않은 popAll 메서드 - 결함이 있다! (183쪽)
//    public void popAll(Collection<E> dst) {
//        while (!isEmpty())
//            dst.add(pop());
//    }

    // 코드 31-4 E 소비자(consumer) 매개변수에 와일드카드 타입 적용 (183쪽)
    public void popAll(Collection<? super E> dst) {
        while (!isEmpty())
            dst.add(pop());
    }

    // 제네릭 Stack을 사용하는 맛보기 프로그램
    // 하위 타입 지원
    public static void main(String[] args) {
        Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);
        numberStack.pushAll(integers);

        Collection<Object> objects = new ArrayList<>();
        numberStack.popAll(objects);

        System.out.println(objects);
    }
}
```

> 저자 왈, extends 라는 키워드가 상황에 완전히 걸맞는 거 같지는 않다고 한다. 왜냐하면, 하위 타입이란 자기 자신도 포함하지만 그렇다고 자신을 확장한 것은 아니기 때문이란다.


### PECS : producer-extends, consumer-super
- 유연성을 극대화하려면, 원소의 생산자나 소비자용 입력 매개변수에 와일드 카드 타입을 활용한다.
    - 그러나, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면, 지양한다. 타입을 정확히 지정하는 것이 알맞다.

#### 생산자 producer
```java
<? extends T>
```
#### 소비자 consumer
```java
<? super T>
```
> 나프탈린과 와들러는 이를 겟풋 원칙이라고 부른다고 한다.

### 반환 타입
- 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다. 
- 유연성은 커녕 클라이언트 코드에서도 와일드 카드 타입을 써야 하기 때문이다.

### 목표 타이핑 target typing
```java
// 코드 30-2의 제네릭 union 메서드에 와일드카드 타입을 적용해 유연성을 높였다. (185-186쪽)
public class Union {
    public static <E> Set<E> union(Set<? extends E> s1,
                                   Set<? extends E> s2) {
        Set<E> result = new HashSet<E>(s1);
        result.addAll(s2);
        return result;
    }

    // 향상된 유연성을 확인해주는 맛보기 프로그램 (185쪽)
    public static void main(String[] args) {
        Set<Integer> integers = new HashSet<>();
        integers.add(1); 
        integers.add(3); 
        integers.add(5); 

        Set<Double> doubles =  new HashSet<>();
        doubles.add(2.0); 
        doubles.add(4.0); 
        doubles.add(6.0); 

        Set<Number> numbers = union(integers, doubles);

//      // 코드 31-6 자바 7까지는 명시적 타입 인수를 사용해야 한다. (186쪽)
//      Set<Number> numbers = Union.<Number>union(integers, doubles);

        System.out.println(numbers);
    }
}

```
- 자바 8부터 지원한다.
- 이 이전 버전에서는 컴파일러가 올바른 타입을 추론하지 못한다면, 언제든 명시적 타입 인수를 사용해서 타입을 알려준다.

### Comparable

- 입력 매개변수 : E 인스턴스를 생산
- Comparable<E> : E 인스턴스 소비, 언제나 소비자

```java
// 와일드카드 타입을 사용해 재귀적 타입 한정을 다듬었다. (187쪽)
public class RecursiveTypeBound {
    public static <E extends Comparable<? super E>> E max(
        List<? extends E> list) {
        if (list.isEmpty())
            throw new IllegalArgumentException("빈 리스트");

        E result = null;
        for (E e : list)
            if (result == null || e.compareTo(result) > 0)
                result = e;

        return result;
    }

    public static void main(String[] args) {
        List<String> argList = Arrays.asList(args);
        System.out.println(max(argList));
    }
}
```
- Comparable을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 경우에 사용할 수 있다.
    - 예를 들어, SceduledFuture 와 같이 상위 인터페이스인 Delayed는 Comparable을 구현하였으나, 자기 자신은 구현하지 않은 경우 지원할 수 있다.


## 와일드 카드 활용
- 메서드 선언에 타입 매개변수가 한 번만 나오면, 와일드 카드로 대체하라.
    - 특히, public api 인 경우
- 다만, 와일드 카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하는 식으로 활용할 수 있다.
    - 예를 들어, List<?> 에는 null 외에 어떤 값도 넣을 수 없다.
    - 실제 타입을 알아내기 위해서 도우미 메서드는 제네릭 메서드로 구현한다.

```java
// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드 (189쪽)
public class Swap {
    // public static <E> void swap(List<E> list, int i, int j)
    public static void swap(List<?> list, int i, int j) {
        swapHelper(list, i, j);
    }

    // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
    private static <E> void swapHelper(List<E> list, int i, int j) {
        list.set(i, list.set(j, list.get(i)));
    }

    // 클라이언트 코
    public static void main(String[] args) {
        // 첫 번째와 마지막 인수를 스왑한 후 결과 리스트를 출력한다.
        List<String> argList = Arrays.asList(args);
        swap(argList, 0, argList.size() - 1);
        System.out.println(argList);
    }
}
```

- 다소 복잡한 구현이지만, **클라이언트 입장에서는 훨씬 좋다.**
- 클래스 사용자가 와일드 카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 것이다.

### 매개변수(parameter)와 인수(argument)의 차이
- 매개변수는 메서드 선언에 정의한 변수
- 인수는 메서드 호출 시 넘기는 실젯값
# 아이템 26. 로 타입은 사용하지 말라

## 📌 용어**정리**

| 한글 용어 | 영문 용어 | 예 |
| --- | --- | --- |
| 매개변수화 타입 | parameterized type | List<String> |
| 실제 타입 매개변수 | actual type parameter | String |
| 제네릭 타입 | generic type | List<E> |
| 정규 타입 매개변수 | formal type parameter | E |
| 비한정적 와일드카드 타입 | unbounded wildcard type | List<?> |
| 로 타입 | raw type | List |
| 한정적 타입 매개변수 | bounded type parameter | <E extends Number> |
| 재귀적 타입 한정 | recursive type bound | <T extends Comparable<T>> |
| 한정적 와일드카드 타입 | bounded wildcard type | List<? extends Number> |
| 제네릭 메서드 | generic method | static <E> List<E> asList(E[] a) |
| 타입 토큰 | type token | String.class |

## **제네릭을 사용하는 이유**

- 컴파일 시점에서 타입을 체크함으로써 **타입 안정성**을 제공함
- **타입체크와 형변환을 생략**함으로써 코드가 간결해짐

```java
public class Item26 {

    public static void main(String[] args) {
        List<Integer> list = new ArrayList();
        list.add(10);
        list.add(20);
        list.add("30");//컴파일 에러 발생Integer i = (Integer)list.get(2);

        System.out.println(list);
    }
}
```

> java: incompatible types: java.lang.String cannot be converted to java.lang.Integer
> 

## **로 타입**

**로 타입(raw type)이란?**

- **제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 경우**를 의미함
- 예) List<E>의 로 타입은 List다

**로 타입을 사용하면 안 되는 이유**

- 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작한다
- 즉, 컴파일 시점에서 타입을 체크하지 않고 런타임 시점에서 타입을 체크한다

```java
public class Item26 {

    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(10);
        list.add(20);
        list.add("30");

        Integer i = (Integer)list.get(2);//런타임 에러 발생

        System.out.println(list);
    }
}
```

> Exception in thread "main" java.lang.ClassCastException: class java.lang.String cannot be cast to class java.lang.Integer (java.lang.String and java.lang.Integer are in module java.base of loader 'bootstrap') at item26.Item26.main(Item26.java:13)
> 
- 에러는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다
- 위 예시에서는 에러가 발생하고 한참 뒤인 런타임에서야 알아챌 수 있다
    - 이렇게 되면 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 커진다
    - 따라서 디버깅이 힘들어진다
- 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다
    - 안정성) 컴파일 시점에서 타입을 체크한다
    - 표현력) 특정 타입의 인스턴스를 사용한다는 정보가 주석이 아닌 타입 선언 자체에서 명시된다(*List<String>*)
- 로 타입은 제네릭 이전 코드들과의 호환성을 위해서만 사용한다

## **로 타입의 대안**

### **임의 객체를 허용하는 매개변수화 타입**

- List<Object>처럼 임의 객체를 허용하는 매개변수화 타입
- 타입 안정성을 보장한다

**로 타입 List를 쓴 경우**

```java
public class Item26 {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);// 컴파일러가 자동으로 형변환 코드를 넣어준다, 런타임 에러 발생
    }

    private static void unsafeAdd(List list, Object o) {
        list.add(o);// 경고 발생
    }
}
```

> Unchecked call to 'add(E)' as a member of raw type 'java.util.List'
> 
- 위 코드는 컴파일이 되지만 다음과 같이 경고가 발생한다
- 프로그램을 실행하면 ClassCastException이 발생한다

**임의 객체를 허용하는 매개변수화 타입 List<Object>를 쓴 경우**

```java
public class Item26 {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));// 컴파일 에러 발생String s = strings.get(0);// 컴파일러가 자동으로 형변환 코드를 넣어준다
    }

    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }
}
```

> java: incompatible types: java.util.List<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
> 
- 위와 같은 컴파일 에러가 발생하여, 컴파일 조차 되지 않는다

### **비한정적 와일드카드 타입**

- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않은 경우
- 로 타입 대신 물음표(?)를 사용한다

Set에 어떤 원소가 들어올 지 모르는 상황에서 다음과 같이 코드를 작성할 수 있다

```java
public int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```

- 이 메서드는 동작은 하지만 로 타입을 사용해 안전하지 않다
- 또한, 아무 원소나 넣을 수 있으므로 타입 불변식을 훼손하기 쉽다

```java
public int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```

- 다음과 같이 비한정적 와일드카드 타입을 사용해 더욱 안전하게 코드를 작성할 수 있다
- Collection<?>에는 null 외엔 어떤 원소도 넣을 수 없으므로 컬렉션의 타입 불변식을 훼손하지 못한다

## **로 타입을 써도 되는 경우(예외)**

- **class 리터럴에서는 로 타입을 써야 한다**
    - 자바 명세에서 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다(배열과 기본 타입은 허용)
    - List.class, String[].class, int.class는 허용한다
    - List<String>.class, List<?>.class는 허용하지 않는다
- **instanceof연산자**
    - 로 타입이든 비한정적 와일드카드 타입이든 instanceof는 완전히 똑같이 동작함

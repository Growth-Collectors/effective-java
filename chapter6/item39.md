# 아이템 39. 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴의 단점

- 오타가 나면 안된다. 예를 들어 JUnit3를 사용할 경우 실수로 이름을 tsetSafetyOverride로 지으면 개발자는 이 테스트가 통과했다고 오해할 수 있다.
- 올바른 프로그램 요소에서만 사용되리라는 보장할 수 없다. 클래스 이름에 Test를 붙여서 지을 경우 개발자는 JUnit이 테스트를 실행해줄꺼라고 생각했지만 그러지 않는다.
- 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다. 특정 예외를 던져야만 성공하는 테스트가 있고 기대하는 예외 타입을 테스트에 매개변수로 전달해야 하는 상황이 그 예이다.

### 애너테이션 도입

- 애너테이션은 이 모든 문제를 해결해주는 개념으로, Junit도 4부터 전면 도입하였다.
- Test라는 이름의 애너테이션을 정의한다. 자동으로 수행되는 간단한 애너테이션으로, 예외가 발생하면 해당 테스트를 실패로 처리한다.
- @Retention과 @Target인데 애너테이션 선언에 다른 애너테이션을 메타애너테이션 이라 한다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)// @Test가 런타임에도 유지되어야 한다는 표시이다.
@Target(ElementType.METHOD)// @Test가 반드시 메서드 선언에서만 사용돼야 한다.
public @interface Test {

}
```

@Test 어노테이션을 적용한 클래스를 작성해보자.

```java
public class Sample {

    @Test
    public static void m1(){}//성공public static void m2(){}

    @Test
    public static void m3(){
        throw new RuntimeException("실패");
    }

    public static void m4(){}

    @Test
    public void m5(){//잘못 사용한 예. 정적 메서드가 아니다.

    }

    public static void m6(){}

    @Test
    public static void m7(){
        throw new RuntimeException("실패");
    }

    public static void m8(){}

}
```

이제 이 클래스의 메서드를 테스트하는 소스 코드를 작성해보자.

Sample 클래스에서 @Test 어노테이션이 붙어있지 않아도 테스트 대상에 잡히지 않는다.

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class RunTests {

    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("ch6.hoon.item39.Sample");
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }

}
```

### 특정 예외를 던져야 성공하는 테스트

다음은 특정 예외를 던져야만 성공하는 테스트를 지원하도록 해보자. 그러려면 새로운 애너테이션 타입이 필요하다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

이 애너테이션의 매개변수 타입에서 와일드카드 타입은 많은 의미를 담고 있다.
"Throwable을 확장한 클래스의 Class객체"라는 뜻이며, 따라서 모든 예외와 타입을 다 수용한다.

```java
Class<? extends Throwable> value();
```

이 어노테이션을 적용한 클래스를 만들자.

```java
public class Sample2 {

    @ExceptionTest(ArithmeticException.class)
    public static void m1() {// 성공해야 한다.int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() {// 실패해야 한다. (다른 예외 발생)int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }// 실패해야 한다. (예외가 발생하지 않음)

}
```

그리고 이를 테스트하는 코드도 작성한다. 코드를보면 예외가 발생할 경우 passed의 숫자를 증가시켜주고 있다.

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ExceptionRunTest {

    public static void main(String[] args) throws Exception {

        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName("ch6.hoon.item39.Sample2");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }

        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }

}
```

위의 테스트 코드에서 좀 더 나아가서 예외를 여러개 명시하고 그중 하나가 발생하면 테스트가 성공인걸로 만들수도 있다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MultiExceptionTest {
    Class<? extends Exception>[] value();
}
```

배열 매개변수를 받는 애너테이션용 문법은 아주 유용하다. 단일 원소 배열에 최적화했지만, 앞서의 @ExceptionTest들도 모두 수정 없이 수용한다.

```java
@MultiExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {// 성공해야 한다.
    List<String> list = new ArrayList<>();

// 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나// NullPointerException을 던질 수 있다.
    list.addAll(5, null);
}
```

### @Repeatable

자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.
배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타 애너테이션을 다는 방식이다.
@Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러 번 달 수 있다.

@Repeatable 주의점

- Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나 더 정의하고,
  @Repeatable에 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
- 컨테이너 에너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 컨테이너 애너테이션@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
		ExceptionTest[] value();
}
```

앞의 배열 방식 대신 반복 가능한 애너테이션을 적용하면 아래 코드와 같다.

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {...}
```

반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높여보였다. 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면 적당한 애너테이션 타입도 함께 정의해 제공하자. 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

하지만 애너테이션을 선언하고 이를 처리하는 부분에서는 코드 양이 늘어나며,
특히 처리 코드가 복잡해져 오류가 날 가능성이 커짐은 명심하자.

도구 제작자를 제외하고는, 일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 거의 없다.
하지만 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.
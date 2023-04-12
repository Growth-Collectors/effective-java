# 아이템 25 : 톱레벨 클래스는 한 파일에 하나만 담으라

### 톱레벨 클래스란?

[오라클](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html)에서는 톱 레벨 클래스를 다음과 같이 정의하고 있다. 

`A top level class is a class that is not a nested class.` 

- 즉, 톱레벨 클래스는 중첩되지 않은 클래스를 의미한다. 
레벨 클래스는 한 클래스에 하나만 작성하도록 해야 한다.

### 💡 예시

`Main` 클래스에서 다음과 같이 `Utensil` 클래스와 `Dessert` 클래스를 참조하고 있다.

```java
/* Main */
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

- `Utensil` , `Dessert` 클래스 모두 톱레벨로 작성.

```java
/* Utensil.java */

class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    
    static final String NAME = "cake";
}
```

`Utensil` 클래스와 `Dessert` 클래스가 모두 정의되어 있기 때문에 결과값이 `pancake` 으로 나온다.

---

- 똑같은 두 클래스를 담은 `Dessert.java`

```java
/* Dessert.java */
class Utensil {
    static final String NAME = "pot";
}
class Dessert {
    
    static final String NAME = "pie";
}
```

이렇게 `Utensil`파일에 있는 똑같은 클래스를 중복 정의했을때, 
컴파일러의 동작 결과가 달라지므로 문제가 발생한다.

- `javac Main.java Utensil.java`: `Main` 클래스에서 참조하는 `Utensil` 클래스와 `Dessert` 클래스는 `Utensil.java`에 명시되어 있는 것이므로 `pancake`가 출력된다.
- `javac Dessert.java Main.java`: `Dessert.java`에 있는 `Dessert` 클래스와 `Utensil` 클래스를 사용하게끔 하므로 `potpie`가 출력된다.
- 이와 같이 컴파일을 어떤 순서대로 하느냐에 따라서 결과가 달라질 수 있는 위험도 존재한다.

---

### 🚨 해결책

- 톱레벨 클래스인 `Utensil` 클래스와 `Dessert` 클래스를 서로 다른 소스 파일로 분리한다.
즉, `Utensil.java`에 `Utensil` 클래스만을 담아두고, `Dessert.java`에 `Dessert` 클래스만을 담아두면 컴파일 중복도 해결되며 컴파일 순서에 따른 다른 결과도 나오지 않게 된다.
- 만약 다른 클래스에 종속적인 클래스들을 한 파일 안에 포함시켜야 한다면 정적 멤버 클래스로 만들어 사용하면 되고, `private` 중첩을 적용시키면 접근 범위도 최소화할 수 있다.

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    private static class Utensil {
        static final String NAME = "pan";
    }
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

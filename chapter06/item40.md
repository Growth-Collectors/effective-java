## item 40 **@Override 애너테이션을 일관되게 사용하라**

추상 메소드를 구현할 때를 제외하고는 `@Override` 를 사용하라! 

### @Override

- 자바에서 기본으로 제공하는 애너테이션
- 상위 타입의 메서드를 재정의 했음을 의미
- 일관되게 사용하면 악명 높은 버그를 예방할 수 있음

### @Override를 사용하지 않는 경우

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```

Set을 사용했으니까 aa ~ zz까지 26이 나올 것이라고 예상

그런데 260이 나왔다!

- equals는 오버라이딩이 아닌 오버로딩을 하고 있었음

@Override를 추가했으면 컴파일 단계에서부터 오류 잡을 수 있었음

```java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```

**상위 클래스의 메서드를 재정의 하는 모든 메서드에 `@Overriding` 애너테이션을 달아주자!**

- 오타, 실수 등을 IDE 및 컴파일 단에서 잡아낼 수 있음  ⇒ 문제를 찾기 위한 시간을 단축 시켜줌

### 핵심정리

- 재정의한 모든 메서드에`@Overriding` 를 의식적으로 달면 실수했을 때 컴파일러가 바로 알려줄 것임
- 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우에는 애너테이션 달지 않아도 됨

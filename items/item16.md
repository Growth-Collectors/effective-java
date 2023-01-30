## [Item 16] public 클래스에는 public 필드가 아닌 접근자 메서드를 사용하라

- 인스턴스 필드를 모아놓는 일 외에는 목적이 없는 퇴보한 클래스  → 캡슐화의 이점 제공 x

```java
class Point {
    public double x;
    public double y;
}
```

### 캡슐화

- API를 수정하지 않고 내부 표현을 바꿀 수 있다.
- 불변식을 보장한다.
- 외부 필드에서 접근할 때 부수 작업 수행할 수 있다.

### **접근자(getter)와 변경자(setter) 메서드를 활용한 데이터 캡슐화**

```java
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    **public double getX() {
        return x;
    }

    public void setX(double x) {
        this.x = x;
    }

    public double getY() {
        return y;
    }

    public void setY(double y) {
        this.y = y;
    }**
}
```

- **패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공**함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
- **package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.**
    - 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다.
- **public 클래스의 필드를 직접 노출하지 마라**

### 불변 필드를 노출한 public 클래스 - 과연 좋은가 ?

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 좋지 않음

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    **public final int hour;
    public final int minute;**

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException(" 시간 : " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException(" 분 : " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

- API를 변경하지 않고는 표현 방식을 바꿀 수 없다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다.
- 단, 불변식은 보장할 수 있게 된다.

### 핵심 정리

- public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
- 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
- package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.

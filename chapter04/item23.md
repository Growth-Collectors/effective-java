# item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스
- 두 가지 이상의 의미를 표현할 수 있으며, 현재 표현하는 의미를 태그 값으로 얄려주는 클래스
- 단점이 한가득!
- 오류를 내기 쉽고 비효율적이다.
- 클래스 계층구조를 어설프게 흉내 낸 아류
```java
package effectivejava.chapter4.item23.taggedclass;

// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다! (142-143쪽)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

### 단점
1. 열거 타입 선언, 태그 필드, switch문 등 쓸데 없는 코드가 많다.
2. 여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다.
3. 다른 의미를 위한 코드도 함께 하여, 메모리도 많이 사용한다.
4. 필드들을 final로 선언하려면, 해당 의미에서 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.
    - 쓰지 않는 필드를 초기화하는 불필요한 코드 증가
5. 해당 의미에 쓰이는 데이터 필드를 초기화하는데 어려움이 있다. 
    - 엉뚱한 필드를 초기화해도 런타임에 문제가 드러난다.
6. 다른 의미를 추가하려면 코드를 수정해야 한다.
7. 인스턴스의 타입만으로는 현재 클래스가 나타내는 의미를 알기 어렵다.

## 클래스 계층구조를 활용하는 서브타이핑(subtyping)
- 객체 지향 언어에서 제공하는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단
- 태그 달린 클래스의 단점 모두 해소
    - 간결하고 명확하며 쓸데없는 코드 모두 제거
    - 각 의미를 독립된 클래스에 담아, 관련 없던 필드를 모두 제거
        - 살아남은 필드는 모두 final
    - 클래스의 생성자가 모든 필드를 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인할 수 있다.
    - 타입 사이의 자연스러운 계층 관계를 반영하여, 유연성은 물론 컴파일 타임 검사 능력을 높여준다는 장점이 있다.
    - 루트 클래스 코드를 건들지 않고, 독립적으로 계층구조를 확장하여 함께 사용 가능
    - 타입이 의미별로 따로 존재하여, 변수의 의미를 명시하거나 제한할 수 있고, 특정 의미만 매개변수로 받을 수 있다.

### 활용 
```java
// 이하 코드에는 접근자 메서드가 생략되어 있다. 
//접근자 활용을 권장하고 아래와 같이 필드를 직접 노출하는 것은 지양한다.(아이템 16)
package effectivejava.chapter4.item23.hierarchy;

// 코드 23-2 태그 달린 클래스를 클래스 계층구조로 변환 (144쪽)
abstract class Figure {
    abstract double area();
}

package effectivejava.chapter4.item23.hierarchy;

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

package effectivejava.chapter4.item23.hierarchy;

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

package effectivejava.chapter4.item23.hierarchy;

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```
1. 계층 구조의 루트(root)가 될 추상 클래스를 정의
    -  태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언
    - 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가
    - 모든 하위 클래스에서 공통으로 사용하는 데이터 필드들도 전부 루트 클래스로 이전
2. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의
    - 각자의 의미에 해당하는 데이터 필드들을 추가
    - 루트 클래스가 정의한 추상 메서드들을 각자의 의미에 맞게 구현

# item13. clone 재정의는 주의해서 진행하라

## Cloneable
- 복제해도 되는 클래스임을 명시하는 용도의 **믹스인 인터페이스** (아이템 20)
- 메서드를 갖지 않는다.

### 특이점(단점)
- Cloneable 에서 활용하는 **clone() 메서드**는 **Object**에 **protected**로 선언되어 있다.
    - 외부에 접근이 허용된 clone() 메서드가 없는 경우엔 리플렉션을 통해 사용이 가능하긴 하나, 항상 그런 것은 아니다.
    - 보편적으로 실무에서 Cloneable을 구현한 클래스는 clone 메서드를 public으로 제공한다.
- 인터페이스를 활용한 상당히 이례적인 예이므로 따라하지 않는다.
    > 일반적으로 인테페이스를 구현하는 것은 구현 클래스에서 인터페이스에서 정의한 기능을 제공한다고 선언하는 것
    - 이에 반해 Cloneable 은 상위 클래스에 정의된 protected clone() 메서드의 동작 방식을 변경해서 활용한다.
- 깨지기 쉽고, 위험하고, 모순적인 메커니즘
    - 생성자를 호출하지 않고 객체를 생성한다.

### clone 메서드의 일반 규약
- 객체의 복사본을 생성해 반환한다.
    - 복사의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다.
- 아래의 식은 참이다. 그러나 반드시 만족해야 하는 것은 아니다.
    - x.clone() != x
        - 복사본은 원본과 같은 인스턴스가 아니다.
    - x.clone().equals(x)
        - 동치성을 만족한다.
    - x.clone().getClass() == x.getClass()
        - 인스턴스의 타입은 같다.
        - 관례상, clone 메소드가 반환하는 객체는 super.clone을 호출해서 얻는다.
            - (Object를 제외한) 모든 상위 클래스가 이 관례를 따르면, 위 식은 항상 참이다. <--- **TODO 이해 안됨**
        - 관례상, 반환된 객체와 원본 객체는 독립적이다.
            - 이를 만족하려면 super.clone 으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

> 관례상, clone 메소드가 반환하는 객체는 **super.clone을 호출**해서 얻는다.
- 강제성이 없다는 점을 빼면 **생성자 연쇄**와 유사한 메커니즘
    - 클래스가 final 이라면, 걱정해야 할 하위 클래스가 없으니, 이 관례는 무시할 수 있다.
        - 사실 final 클래스의 clone 메서드가 super.clone 을 호출하지 않는다면, Cloneable을 구현할 이유가 없다.

## Cloneable 구현해보기
> - clone 메소드를 통해 얻는 객체는 원본의 완벽한 복제본일 것이다.  
>   - 클래스에 정의된 모든 필드는 원본 필드와 똑같은 값을 갖는다.
> - clone 메서드는 생성자와 같은 효과를 낸다.
>    - clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

#### 1. 클래스에 정의된 **모든 필드가 기본 타입이거나 불변 객체를 참조**하는 경우
```java
    // 코드 13-1 가변 상태를 참조하지 않는 클래스용 clone 메서드 (79쪽)
    @Override public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 일어날 수 없는 일이다.
        }
    }
```
- 쓸데없는 복사를 지양한다는 관점에서 불변 클래스는 굳이 clone 메소드를 제공하지 않는다.
- **공변 반환 타이핑**을 활용해 반환 타입을 지정한다.
    - 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.
    - 클라이언트가 형변환 하는 수고를 덜 수 있다.
- Object의 clone 메서드가 **검사 예외**인 CloneNotSupportedException을 던지도록 선언되어 있다.
    - 이로 인해 try-catch 구문으로 감싸고, 예외 발생시 **비검사 예외**를 던지도록 한다. (아이템 71)
    - 불필요한 검사 예외로 번거로운 코드 작성이 발생하게 된다.


#### 2. 클래스가 가변 객체를 참조하는 경우
- 내부 정보를 복사하기 위해 clone을 재귀적으로 호출한다.
```java
// Stack의 복제 가능 버전 (80-81쪽)
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size ==0;
    }

    // 코드 13-2 가변 상태를 참조하는 클래스용 clone 메서드
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
//... 생략
}    
```
> - 배열의 clone은 런타임 타입과 컴파일 타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다.
>   - 배열을 복제할 때는 clone 메소드의 사용을 권장한다.
> - Cloneable 아키텍쳐는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.
>   - 원본과 복제된 객체가 final 가변 객체를 공유해도 안전하다면 괜찮다.
>   - 가변 객체의 필드가 final 인 경우, 새로운 인스턴스를 전달할 수 없다.

#### 3. 복잡한 가변 객체 복제 - clone을 재귀적으로 호출하는 것만으로는 어려운 경우
1. 클래스 자체적으로 **깊은 복사**를 지원하도록 보강
    - 깊은 복사에서 재귀 호출로 인한 성능 문제
        - 예를 들어 연결 리스트 복제시, 리스트의 원소 수만큼 스택 프레임을 소비하게 된다. 
        - 리스트가 긴 경우 스택 오버 플로우 발생 위험 존재하므로 **재귀 호출 대신 반복자**를 써서 순회하는 방식으로 해결한다.
2. 고수준 API 활용해서 복제
    1. super.clone 을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한다.
    2. 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출한다.
    - 저수준에서 바로 처리할 때보다 느리다.
    - Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하므로, 어울리는 방식은 아니다.  
> 생성자에서는 재정의 될 수 있는 메서드를 호출하지 않아야 한다. (아이템 19)  
>clone 메서드에서도 마찬가지다. 
> - 아니라면, 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.

### 상속용 클래스
- **상속해서 쓰기 위한 클래스 설계 방식 두가지** (아이템 19)
- 상속용 클래스는 Cloneable을 구현해서는 안 된다.
    1. Object 방식을 모방해서 방지
        - 제대로 작동하는 clone 메서드를 구현해 protected로 두고, CloneNotSupportedException도 던질 수 있다고 선언한다.
        - 하위 클래스에서 선택적으로 Cloneable을 구현할 수 있다.
    2. clone을 동작하지 않게 구현해두고 하위 클래스에서 재정의하지 못하게 한다.
        ```java
        // 퇴화시킨 clone 메서드
        @Override
        protected final Object() clone() throws CloneNotSupportedException {
            throw new CloneNotSupportedException();
        }
        ```

### 스레드 안전
- clone을 재정의하여 동기화해줘야 한다.

## Cloneable 요약
- Cloneable을 구현하는 모든 클래스는 **clone을 재정의**한다.
    - 접근 제한자는 public으로 반환 타입은 클래스 자신으로 변경한다.(공변 반환 타이핑)
    - super.clone 호출 후, 필요한 필드를 전부 적절히 수정한다.
    - 객체 내부 **깊은** 구조에 숨어 있는 모든 **가변 객체**를 복사하고, 복제본이 가진 **객체 참조 모두가 복사된 객체**를 가리키게 한다.
        - 내부 복사는 주로 clone 을 재귀적으로 호출해서 구현하지만, 항상 최선인 것은 아니다.
    - 기본 타입 필드와 불변 객체 참조만 갖는 경우에는 수정할 필요가 없다.
        - 그러나, 일련번호나 고유 ID는 수정해준다.
- Cloneable을 이미 구현한 클래스를 확장하는 경우에 사용한다. (어쩔 수 없이)
- final 클래스의 경우에는 구현할 수 있겠지만, 성능 최적화 관점에서 검토한 후 결정한다.
- **오직 배열**만이 clone 메서드 방식이 가장 깔끔하므로 활용한다.

### 대안) 변환(복사) 생성자와 변환(복사) 팩터리
- 더 나은 객체 복사 방식을 제공
- 복사 생성자 : 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자 == 변환 생성자
- 복사 팩터리 : 복사 생성자를 모방한 정적 팩토리 (아이템 1) == 변환 팩터리
### Cloneable 단점들(이하 나열)을 보완 
- 언어 모순적이고 위험한 객체 생성 메커니즘(생성자 쓰지 않는 방식)
- 엉성하게 문서화 된 규약
- 정상적 final 필드 사용법과 충돌
- 불필요한 검사 예외 사용
- 형변환 필요
### 장점
- 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다.
    - 예를 들어, 관례상 모든 범용 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다.
    - 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 지정할 수도 있다.
        - 예를 들어, HashSet 객체 -> TreeSet 타입 객체로 복제 가능하다.

## 결론
-  **새로운 인터페이스를 만들 때 절대 Cloneable을 확장하지 않으며, 새로운 클래스도 이를 구현하지 않는다.**
- **복제 기능은 생성자와 팩터리를 이용하는 것이 최선이다.**
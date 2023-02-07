## [Item18] ****상속보다는 컴포지션을 사용하라****

### Intro

- **상속**은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.
    - 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.
- 상위 클래스, 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법일 수 있다.
    - 하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.

### **잘못된 예 - 상속을 잘못 사용했다!**

HashMap 기능을 사용하면서, 생성된 이후 몇개의 원소가 더해졌는지 알 수 있는 기능을 추가한 클래스를 구현

```java

public class InstrumentedHashSet<E> extends HashSet<E> {
  private int addCount = 0;
  
  public InstrumentedHashSet(int initCap, float loadFactor) {
  super(initCap, loadFactor);
  }

  @Override 
  public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

  @Override 
  public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("용", "용용", "용용용"));
```

`getAddCount` 를 호출하면 3이 반환될 것 같지만 **실제로는 6을 반환한다.**

- HashSet의 `addAll` 은 각 원소마다 `add` 를 호출하게 구현이 되어있다.
- 이처럼 오류를 범할 수 있으며 직접 `HashSet`의 `addAll()` 메서드를 확인하기 전까진 그 이유를 알 수 없습니다.

```java
public abstract class AbstractCollection<E> implements Collection<E> {
    // ...

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    // ...
}
```

이 예제의 경우, addAll 메서드를 재정의하지 않거나, 다른 식의 재정의를 통해 문제를 해결할 수 있다.

- **재정의 하지 않는 경우 (HashSet의 `addAll` 을 사용하는 경우)**
    - HashSet의 addAll메서드가 add메서드를 이용해 구현했다는 것을 가정한다는 한계를 가진다.
    - 즉, **현재 addAll 메서드의 구조에만 의존**하게 되는 것이다. → 구조 변화가 일어나면 문제가 생길 것
- **다른 식의 재정의를 하는 경우 (InstrumentedHashSet에서 아예 새롭게 `addAll` 을 재정의 하는 경우)**
    - 상위 클래스 메서드와 똑같이 동작하도록 구현해야 하는데, 이 방식은 어**려울 수도 있으며, 시간도 더 들고, 오류 및 성능하락의 문제**를 가져올 수 있다.

### 다음 릴리스에서 상위 클래스에 새로운 메서드를 추가한다면?

- 새롭게 추가된 메서드를 통해서 허용되지 않은 동작을 수행할 수 있게 될 수도 있다.
- 만약 하위 클래스에 추가한 메서드와 상위 클래스에 새롭게 추가된 메서드의 시그니처가 같고 반환 타입이 다르다면 컴파일조차 되지 않는다.
    - 하위 클래스에 메서드를 작성할 때는 상위 클래스의 메서드는 존재하지도 않았으니, 하위 클래스의 메서드는 상위 클래스에 새롭게 추가된 메서드가 요구하는 규약을 지키지 못할 가능성이 크다.

## 문제들을 해결할 좋은 방법이 있다!

- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.
    - 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 **컴포지션**(composition; 구성)이라 한다.
- 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.
    - 이 방식을 **전달(forwarding)**이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.
    - 그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

## 컴포지션이란?

기존 클래스를 확장하지 않고, 새로운 클래스를 만든 후, private 필드로 기존 클래스의 인스턴스를 참조하게 하도록 설계하는 방법이다.

**→ 기존 클래스가 새로운 클래스의 구성요소로 쓰이게 되는 구조**

### **HashMap 예제를 컴포지션으로 변환**

```java
public class InstrumentedHashSetUseComposition<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedHashSetUseComposition(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

### 아래는 재사용할 수 있는 전달클래스인 ForwardingSet

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public int size() {
        return 0;
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public void clear() {
        s.clear();
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
```

### 용어

- **전달(Forwarding)**
    - 새 클래스의 메서드들이 기존 클래스를 대응하는 메서드들을 호출해 그 결과를 반환하는 것
- **전달 메서드(Forwarding method)**
    - 전달(Forwarding)을 수행하는 새로운 클래스의 메서드들
- **래퍼 클래스 (Wrapper class), 데코레이터 패턴 (Decorator pattern)**
    - InstrumentedSet 같이 Set 인스턴스를 감싸고, 기능을 덧씌운다는 뜻에서 이렇게 불린다.

### 컴포지션의 장점

- 한 번만 구현해두면 사용한 인터페이스(Set, Map, ...)의 어떠한 구현체에도 적용이 가능하다.
- 기존 클래스 내부 구현방식의 영향에서 벗어나며, 기존 클래스의 새로운 메서드가 추가되더라도 전혀 영향받지 않는다. (유연하다)
- 단점이 거의 없다

### 컴포지션의 단점

- 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다.
- 콜백(Callback) 프레임워크와는 어울리지 않는다.

### 래퍼 클래스와 SELF 문제

- 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다.
- 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 되는데, 이를 SELF 문제라고 한다.

## 상속과 is-a

- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.
- 즉, 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
- 만약 is-a 관계가 아니라면 A는 B의 필수 구성요소가 아니라 구현하는 방법 중 하나일 뿐이다.
- is-a는 말 그대로 'A는 B이다'일 때의 '~이다'와 같다.

## 핵심 정리

- 상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
- 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다.
    - is-a 관계일 때도 안심할 수만은 없다. 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았을 수도 있기 때문이다.
- 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용한다. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.

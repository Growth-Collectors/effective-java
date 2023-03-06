## item 41 ****정의하려는 것이 타입이라면 마커 인터페이스를 사용하라****

### 마커 인터페이스 (marker interface)

- 어떠한 메서드 선언이 없으며, 단지 클래스를 표시하기 위해 사용되는 인터페이스
- Cloneable, Serializable 같은 인터페이스

예시)

`Serializable` 인터페이스를 구현한 클래스는 ObjectOutputStream으로 작성될 수 있는 인스턴스를 뜻하게 된다

```java
/*
 * ...
 * @author  unascribed
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Externalizable
 * @since   1.1
 */
public interface Serializable {
}
```

### 마커 어노테이션

- 해당 요소가 **특정 속성**을 가짐을 나타내는 에너테이션
- @Override, @FunctionalInterface, @SafeVarargs, @Native

### 마커 어노테이션 장점

- 스프링 부트와 같은 어노테이션 기반 프레임 워크에서 **일관성** 유지 가능
- **필드나 메서드**에서는 마크 인터페이스 사용할 수 없으니, 마커 어노테이션 사용 해야함

### 마커 인터페이스 장점

- 타입 정의 가능
    - 컴파일 타임 안정성을 말한다
    - ex) 자바 `Serializable` 마커 인터페이스는 해당 타입이 직렬화 가능함(Serializable)을 알려줌
- 타겟 정확하게 지정 가능
    - 특정 인터페이스와 구현체에만 적용 가능한 마커를 가지고 싶으면 마커 인터페이스로 구현하면 됨
    - ex) `java.util.Set` 인터페이스

---

### 핵심 정리

**마커 어노테이션**

- 클래스나 인터페이스를 제외한 프로그램 요소에 마킹 (필드나 메서드)

**마커 인터페이스**

- 새로 추가하는 메서드 없이 단지 **타입** 정의가 목적

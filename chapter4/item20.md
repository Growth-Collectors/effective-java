### 추상 클래스보다 인터페이스를 우선

---

- 자바가 제공하는 다중 구현 메커니즘 → 인터페이스, 추상 클래스
- 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있음
- 둘의 가장 큰 차이
    - **추상 클래스의 경우**, 추상 클래스가 정의한 타입을 구현하는 클래스는 **반드시 추상 클래스의 하위 클래스가 되어야 같은 타입으로 취급**
    - **인터페이스의 경우**, 인터페이스에서 선언한 메서드를 모두 정의한 클래스라면 **다른 어떤 클래스를 상속했든 같은 타입으로 취급**

- 인터페이스
    - 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있음
    - 인터페이스는 믹스인 정의에 알맞음
        - 추상클래스는 기존클래스에 덧쓸 수 없기 때문에 믹스인 정의 불가능
    - 계층구조가 없는 타입 프레임워크를 만들 수 있다
    - 가수와 작곡가 그리고 가수겸 작곡가와 같은 계층적으로 표현하기 어려운 개념은 인터페이스로 구현
        
        ```java
        public interface Singer{
          public void sing();
        }
        
        public interface SongWriter{
          public void compose();
        }
        
        public class People implements Singer, SongWriter{
          @Override
          public void sing() {}
          @Override
          public void compose() {}
        }
        
        public interface SingerSongWriter extends Singer, SongWriter{
          public void actSensitive();
        }
        ```
        
        같은 내용을 추상 클래스로 구현시
        
        ```java
        public abstract class Singer {
            abstract void sing();
        }
        public abstract class SongWriter {
            abstract void compose();
        }
        
        public abstract class SingerSongWriter {
            abstract void actSensitive();
            abstract void sing();
            abstract void compose();
        }
        ```
        
        추상 클래스를 두 개 이상의 클래스를 상속할 수 없기 때문에, 
        SingerSongWriter라는 추상 클래스를 만들어 추상 메소드를 추가할 수밖에 없다
        

### 추상 골격 구현 클래스

---

- 인터페이스와 추상 골격 구현 클래스를 함께 제공해 두 장점을 모두 취하는 방법도 있다

인터페이스: 타입 정의, 디폴트 메서드 몇 개 + 추상 골격 클래스: 나머지 메서드들 구현

- **인터페이스에서는 사용할 메서드를 정의하고, 기반 메서드를 선정한 후 이를 묶어 디폴트 메서드로 제공한다**
    - 하나의 작업을 수행하는 데 필요한 메서드들과, 실행순서를 정해서 **공통의 메서드로 묶는다**
- **추상 클래스에서는 구체 클래스에서 공통적으로 사용되는 메서드들을 재정의하여 중복을 제거한다**
    - **메서드의 구현이 중복되는 부분을 재정의**한다
- **구체 클래스에서는 기반 메서드를 재정의한다**
    - 공통적인 작업의 순서(인터페이스)와 구현(추상 클래스)이 채워졌으니, **개별적으로 동작할 메서드를 재정의**해주면 된다

골격 구현 클래스

```java
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {

    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다. 
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Map.Entry.equals의 일반 규약을 구현한다. 
    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Object.equals(e.getKey(), getKey())
            && Objects.equals(e.getValue(), getValue());
    }

    //Map.Entry.hashCode의 일반 규약을 구현한다. 
    @Override
    public int hashCode() {
        return Objects.hashCode(getKey())
            ^ Objects.hashCode(getValue());
    }

    @Override
    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

3줄 정리

- 다중 구현용 타입으로 인터페이스가 가장 적합
- 복잡한 인터페이스라면 골격 구현을 함께 제공하는 방법을 고려할 것
- 골격 구현은 되도록 인터페이스의 디폴트 메서드로 제공하여, 그 인터페이스를 구현한 모든 곳에서 활용

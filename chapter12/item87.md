### 기본 직렬화 형태를 그대로 사용할 때 고려해야 할 점
- 기본 직렬화 형태는 유연성, 성능, 정확성, 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.
- 기본 직렬화 형태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다.
- 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며 객체들이 연결된 위상(topology)까지 기술한다.
- 하지만 이상적인 직렬화 형태는 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.
- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방함

### 기본 직렬화 선택에 적합한 경우 / 그렇지 않은 경우
- 기본 직렬화 형태는 객체가 포함된 데이터와 객체 주변의 관계들도 나타냄
- 그러나 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 함
- 객체의 물리적 표현 = 논리적 내용 => 기본 직렬화 써도 괜찮음
- 객체의 물리적 표현 != 논리적 내용 => 기본 직렬화 쓰면 안됨
```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 함.
     * @serial
     */
    private final Stirng lastName;

    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;

    ... // 나머지 코드는 생략
}
```
- 성명은 논리적으로 성, 이름, 중간 이름이라는 3개의 문자열로 구성하는데, 위 클래스의 인스턴스 필드들은 이 논리적인 구성 요소를 정확하게 반영함 => 기본 직렬화 써도 ok
```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... // 나머지 코드는 생략
}
```
- 논리적 (문자열을 표현함) + 물리적 (문자열들을 이중 연결 리스트로 표현함)
   - 물리적: 객체를 둘러싼 컨텍스트를 의미하는듯?
 - 기본 직렬화 형태를 사용한다면 next, previous 로 연결된 노드들까지 모두 표현할 것이다 => 기본 직렬화 선택 no

### 객체의 물리적 표현과 논리적 표현의 차이가 클 때 생기는 문제
- 공개 API가 그 논리적 컨텍스트에 종속적이게 됨
   - 예를 들어, 향후 버전에서는 연결 리스트를 사용하지 않게 바꾸더라도 관련 처리는 필요해진다. 따라서 코드를 절대 제거할 수 없다.
  - 논리적 표현을 직렬화하는데 너무 많은 공간을 차지함
    - 논리적 컨텍스트는 내부 구현일 뿐인데, (직렬화에 가치가 없음) 네트워크 전송량만 많아짐
  - 스택 오버플로를 발생시킴
    - 내부 구현체 재귀 순회하느라 스택이 쌓이고 쌓여 오버플로 위험 있음

### 합리적인 직렬화 형태
- 단순히 리스트가 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 수준이면 되겠다.
- StringList의 상세 관계들은 빼고, 구성만 담는다
- writeObject (직렬화 담당), readObject (역직렬화 담당)
- 일시적이란 뜻의 transient한정자: 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시
```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다. -> 
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 지정한 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            stream.writeObject(e.data);
        }
    }
    
    /**
     * 역직렬화 담당 
     */
    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    ... // 나머지 코드는 생략
}
```
- StringList의 필드 모두가 transient더라도 writeObject와 readObject는 각각 가장 먼저 defaultWriteObject, defaultReadObject를 호출한다. 
  - 어차피 defaultWriteObject 는 transient를 직렬화 안할텐데 왜 굳이 호출? 
    - -> 이렇게 해야 향후 릴리즈에서 transient가 아닌 인스턴스 필드가 추가되더라도 상위/하위 호환되기 때문. 
  - 만약 그렇게 해주지 않는다면?
    - 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화하면 새로 추가된 필드들은 무시될 것이다.
    - 구버전 readObject메서드에서 defaultReadObject를 호출하지 않는다면 역직렬화할 때 StreamCorruptedException이 발생할 것이다. 
- transient 키워드
  - transient 키워드를 붙이면 직렬화에서 제외됨
  - 논리적 상태와 무관한 필드라고 판단될 때만 transient 한정자를 생략해야 한다.
  - -> 커스텀 직렬화 형태를 사용한다면, 앞서의 StringList예에서처럼 대부분의 인스턴스 필드를 transient로 선언해야 한다

### 동기화
- 직렬화에도 동기화 규칙을 적용해야 한다
- ex) 모든 메서드가 synchronized로 선언되어 스레드안전하게 선언되어 있다면? writeObject도 아래처럼 수정해야 한다.
```java
private synchronized void writeObject(ObjectOutputStream stream)
        throws IOException {
    stream.defaultWriteObject();
}
```

### UID
- 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬화 버전 UID를 명시적으로 부여하자
- (내가 넣어주지 않는다면 런타임에 이 값이 자동생성될수밖에 없음 -> 성능에 영향을 줌)
- UID가 동일하다 = 직렬화 알고리즘(?)이 동일한 버전이다 = 직렬화 알고리즘이 호환된다
- UID가 다르다 = 역직렬화시 InvalidClassException 에러가 남

### Serializable 단점들
- 클래스 내부구현 못바꿈: Serializable 구현하고 릴리즈하는 순간 그 구조가 공개API가 되어버림
- 캡슐화 깨짐: 자바의 기본 직렬화 형태에서는 private, protected도 공개되도록 되어있음 
- 보안상 안좋음: 직렬화가 있다면 역직렬화도 있게될텐데, 역직렬화는 보안상 나쁨
- 테스트할게 많음: 클래스의 신버전이 릴리즈된다면? 구버전 역직렬화모듈에서 직렬화된 신버전 클래스가 잘 처리되는지 테스트해야함

### Best practice? 
- 역사적으로 Biglnteger와 Instant 같은역사적으로 Biglnteger와 Instant 같은 "값"클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 "동작"하는 객체를 표현하는 클래스들은 대부분 Serializable을 구현하지 않았다

### 주의점
- 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안 됨
  - 상속용 클래스가 Serializable 인터페이스를 구현하면, 클래스의 모든 하위 클래스도 직렬화를 지원하게 됨. 만약 상속용 클래스의 구현이 변경되면, 하위 클래스의 직렬화된 인스턴스와의 호환성이 깨진다
- 인터페이스도 대부분 Serializable을 확장해서는 안 된다. 
  -  인터페이스가 Serializable을 확장하면, 해당 인터페이스를 구현하는 모든 클래스가 직렬화를 지원하게 됨 
- 내부 클래스는 직렬화를 구현하지 말아야 한다.
  - 내부 클래스는 바깥 클래스의 인스턴스에 대한 암묵적인 참조를 갖고 있음. 이 참조는 컴파일러가 자동으로 생성한 필드를 통해 구현됨. 
  - 내부 클래스를 직렬화하면 이렇게 "개발자가 알수없는값"들도 직렬화됨
  - 역직렬화할 때 이 값들은 예상치 못한 행위를 유발할 수 있음

### 예외
- 상속용 클래스더라도 Serializable을 구현할 수밖에 없었던 것들
  - Throwable : RMI(Remote Module Invocation)를 통해 클라이언트로 예외를 보내기 위해 Serializable을 구현했다. (서버에서 예외를 직렬화해서 클라이언트에게 전송함)
  - Component: GUI를 전송하고 저장하고 복원하기 위해 Serializable을 구현했지만, 잘 쓰이지 않음

### 이처럼 상속용 클래스에서 Serializable를 꼭 구현해야한다면?
- 불변식을 보장해야 한다면 반드시 하위 클래스에서 finalize를 재정의하지 못하게 해야한다. (어떻게? finalize를 자신이 재정의하면서 final로 선언함으로써)
  - 안그러면 [finalizer 공격](https://github.com/Growth-Collectors/effective-java/issues/8)  당할 수 있음
- 인스턴스 필드중 불변식은 기본값(정수형은 0, boolean은 false, 객체 참조타입은 null)으로 초기화된다면 위배됨. 이경우엔 클래스에 readObjectNoData메서드를 추가해라.
```java
private void readObjectNoData() throws InvalidObjectException {
    throw new InvalidObjectException("스트림 데이터가 필요합니다.");
}
```
  - 역직렬화 과정에서 `readObjectNoData`가  적절한 값으로 클래스필드를 초기화해줌.
```java
public class Person implements Serializable {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // ... 기타 메서드
}
```
  - 이 클래스가 직렬화되어 들어온다면 에러가 나므로 아래와 같은 로직 구현이 필요
```java
private void readObjectNoData() throws ObjectStreamException {
    this.name = "Unknown";
    this.age = 0;
}
```
# item22. 인터페이스는 타입을 정의하는 용도로만 사용하라

## 인터페이스
- **타입을 정의하는 역할을 수행한다.**
- 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
- 클래스가 어떤 인터페이스를 구현함으로써, 자신의 인스턴스가 어떤 역할을 하는지 클라이언트에게 알려준다.

### 안티패턴 - 상수 인터페이스
```java
package effectivejava.chapter4.item22.constantinterface;

// 코드 22-1 상수 인터페이스 안티패턴 - 사용금지! (139쪽)
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```
- 메서드 없이, 상수를 뜻하는 static field로만 가득 찬 인터페이스
    - 상수들을 사용하려는 클래스에서 정규화 된 이름(qualified name)을 쓰는 걸 피하고자 이러한 인터페이스를 구현
- 상수는 외부 인터페이스가 아닌 내부 구현에 속한다.
- 상수 인터페이스는 내부 구현을 클래스의 API로 노출하는 행위다.
    - 사용자에게는 아무런 의미가 없으며 그저 혼란을 준다.
    - 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 **종속**되게 한다.
    - 해당 상수들을 더 사용하지 않게 되어도 **바이너리 호환성**을 위해 여전히 상수 인터페이스를 유지해줘야 한다.
- final이 아닌 클래스가 상수 인터페이스를 구현한다면, 모든 하위 클래스의 이름 공간이 그 인터페이스가 정의한 상수들로 오염되어 버린다.
- 예시) java.io.ObjectStreamConstants 등

#### 합당한 선택지
1. 특정 클래스나 인터페이스와 강하게 연관된 상수라면, 클래스나 인터페이스 자체에 추가한다.
    - 모든 숫자 기본 타입의 박싱 클래스가 대표적
    - Integer와 Double에 선언된 MIN_VALUE와 MAX_VALUE 상수가 이런 예다.
2. 열거 타입
    - 열거 타입으로 나타내기 적합한 상수라면, 열거 타입으로 만들어 공개한다. (아이템 34)
3. 유틸리티 클래스
    - 인스턴스화 할 수 없도록 만든다.
    ```java
    package effectivejava.chapter4.item22.constantutilityclass;

    // 코드 22-2 상수 유틸리티 클래스 (140쪽)
    public class PhysicalConstants {
    private PhysicalConstants() { }  // 인스턴스화 방지

    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

    // 전자 질량 (kg)
    public static final double ELECTRON_MASS    = 9.109_383_56e-31;
    }
    ```
    - 상수를 클라이언트에서 사용하기 위해 클래스 이름까지 함께 명시해야 한다.
    - 유틸리티 클래스의 상수를 빈번히 사용하는 경우 정적 임포트(static import)하여 클래스 이름을 생략할 수 있다.
    ```java
    import static effectivejava.chapter4.item22.constantutilityclass.PhysicalConstants.*;
    ```

#### 숫자 리터럴과 밑줄_
- 자바 7부터 숫자 리터럴에 밑줄_이 허용된다.
    - 고정소수점, 부동소수점 수 모두 활용 가능하며, 5자리 이상인 경우 이용하는 것을 고려한다.
    - 십진수 리터럴도 밑줄을 사용해 세자리씩 묶어준다.
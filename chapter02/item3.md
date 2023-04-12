## **아이템 3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라**
- 싱글턴 (singleton) 개념
    인스턴스를 오직 하나만 생성할 수 있는 클래스
    - ex)
        - 함수(아이템 24)와 같은 무상태(stateless) 객체
        - 설계상 유일해야 하는 시스렘 컴포넌트

- 한계
클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어
려워질 수 있다.  
(∵ 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체 불가)

- 싱글톤 생성 방법
    - 종류
        - public static 멤버가 final 필드인 방식
        - 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
        - 원소가 하나인 열거 타입을 선언하는 방식
    - 1,2의 공통점  
        생성자는 private
으로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해두는 방식

    - 차이점
        - public static 멤버가 final 필드인 방식
            ```java
            public class Elvis {
            }
            pUblic static final Elvis INSTANCE = new Elvis();
            private Elvis() { ... }
            public void leaveTheBuilding() { ... }
            ```
            - private 생성자는 public static final 필드인 Elvis.IN5TANCE를 초기화할 때 딱
            한 번만 호출된다.
            - public이나 protected 생성자가 없으므로 Elvis 클래스가 초
            기화될 때 만들어진 인스턴스가 전체 시스댐에서 하나뿐임이 보장된다. 
            - 클라이언트는 손 쓸 방법이 없지만 권한이 있는클라이언트는리
            플렉션 API(아이댐 65) 인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다. 이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.
            - 장점
                - 해당 클래스가 싱글턴임이 API에 명백히 드러 난다는 것.(public static 필드가 final이니 절대로 다른 객체를
    참조할 수 없다.)
                - 간결함

            
        - 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
            ```java
            public class Elvis {
            }
            private static f inal Elvis INST때CE = new Elvi s() ;
            private Elvis() { ... }
            public static Elvis getlnstance() { retu rn INSTANCE; }
            public void l eaveTheBuild i n  g() { ... }
            ```
            Elvis.getlnstance는 항상 같은 객체의 참조를 반환하므로 제 2의 Elvis 인스턴스란 결코 만들어 지지 않다(역시 리플렉션을 통한 예외는 똑같이 적용된다)
            - 장점
                - (마음이 바뀌면) API
            를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.  
                유일한 인스턴스를 반환하던 팩터 리 메서드가 (예컨대) 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.  
                - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들수 있다는점(아이템 30).
                - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다  
                ex)`Elvis::getInstance`를 `Supplier<Elvis>`로 사용하는 식이다(아이템 43, 44).
            이러한 장점들이 굳이 필요하지 않다면 public 필드 방식이 좋다.

        - 원소가 하나인 열거 타입을 선언하는 방식
            ```java
            // 코드 3-3 열거 타입 방식의 싱글턴 - 바람직한 방법
            public enum Elvis {
            INSTAl\JCE;
            public void leaveTheBuilding() { .•• }
            }
            ```
            - 특징
                - public 필드 방식과 비슷하지만 더 간결하고 추가 노력 없이 직렬화할 수 있고 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2 의 인스턴스가 생기는 일을 완벽히 막아준다.
                - 조금 부자연스러워 보일 수는 있으나 대부분 상황에서는 원소가 하나뿐인 열거 타업이 싱글턴을 만드는 가장 좋은 방법이다.
                (단， 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다(열거 타입 이 다른 인터페이스를 구현하도록 선언할 수는 있다))
            
- 싱글톤 클래스의 직렬화 방법  
단순히 Serializable을 구현한다고 선언하는 것 뿐만 아니라  
모든 인스턴스 필드를 일시적( transient ) 이라고 선언하고  
readResolve 메서드를 제공해야 한다(아이템 89).
이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다. 코드 3-2 의 예에서라면 가짜 Elvis가 탄생한다는 뜻이다. 이를 예방하기 위해 Elvis 클래스에 다음의 readResolve 메서드를 추가하자.
```java
// 싱글턴입을 보장해주는 readResolve 메서드
private Object readResolve() {
// ’진짜’ El vis를 반환하고 ， 가짜 Elvis는 가비지 컬렉터에 맡긴다.
return INSTANCE;
```

<br>

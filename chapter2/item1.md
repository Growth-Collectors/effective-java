## **아이템 1 - 생성자 대신 정적 패터리 메서드를 고려하라**
- 클라이언트가 클래스의 인스턴스를 얻는 방법
    - 클래스의 public 생성자 이용 - 전통적인 방법
    - 클래스의 정적 팩터리 메서드(static factory method) 이용- 그 클래스의 인스턴스 단순 반환

<br>

- 정적 팩터리 메서드의 장점(정적 팩터리 메서드가 생성자보다 좋은점)
    - 요약
        - 이름을 가질 수 있음
        - 호출될 때마다 인스턴스를 새로 생성하지는 않아도 됨
        - 반환 타입의 하위 타입 객체를 반환할 수 있음
        - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있음
        - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

    - 상세
        - `이름을 가질 수 있음`  
            생성자에 제공하는 인자가 반환하는 객체를 잘 설명하지 못할 경우,  
            잘 만든 이름을 가진 정적 팩토리를 사용하는 것이 사용하기보다 더 쉽고 읽기 좋다.  
            - HOW TO  
            한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면  
            생성자를 정적 팩터리 메서드로 바꾸고  
            각각의 차이를 잘 드러내는 이름을 지어주면 된다.   
            ex) ‘값이 소수인 Biglnteger를 반환한다’는 의미를 가지는 Biglnteger.probablePrime(java 4~)

            - `생성자와의 비교`  
                - 생성자는 시그니츠어 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명 못함.  
                ex) 생성자인 Biglnteger( int, int, Random)
                - 하나의 시그니처로는 생성자를 하나만 만들 수 있다.
                    - 대안
                    입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 방법  
                    => 그런 API를 사용하는 개발자는 각 생성자가 어떤 역할을 하는지 정확히 기억하기 어려워 엉뚱한 것을 호출하는 실수를 할 수 있고 코드 리뷰어들도 클래스 설명 문서를 찾지 않고는 의미를 알지 못할 수 있다.
                
        - `호출될 때마다 인스턴스를 새로 생성하지는 않아도 됨`  
        불변 클래스(immutable class; 아이템 17)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용해서 반복되는 요청에 같은 객체를 반환하는 식으로 불필요한 객체 생성을 피할 수 있다. 
            - 효과
                - 인스턴스 통제 (instance-controlled)
                    - 플라이웨이트 패턴(Gamma95)의 근간이 되며 열거 타입(아이템 34)은 인스턴스가 하나만 만들어짐을 보장한다.
                - 인스턴스 통제 클래스  
                언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있는 정적 팩터리 방식의 클래스
                - 인스턴스 통제하는 이유  
                  - 클래스를 싱글턴(singleton;아이템 3)으로 만들 수 있음
                  - 클래스를 인스턴스화 불가(noninstantiable; 아이댐 4)로 만들 수도 있음
                  - 불변 값 클래스(아이템 17) 에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있음(a == b 일 때만 a.equals(b)가 성립)
                
            ex) Boolean.valueOf(boolean) 메서드  
                객체를 아예 생성하지 않으므로
                (특히 생성 비용이 큰) 같은 객체가 자주 요청되는 상황에서 성능을 상당히 끌어올려 준다.
            - cf) 플라이웨이트 패턴(Flyweight pattern)과 유사  
                인스턴스를 가능한 한 공유해서 사용함으로써 메모리를 절약하는 패턴.
                객체의 특정 옵션까지 동일한 인스턴스가 존재한다면 그걸 전달해주고
                그게 아니면 새로운 객체를 생성해서 전달하는 방식
                - ex) Java의 String Constant Pool  
                자바의 String은 만들어질때 String Constant Pool에 저장되어  
                같은 문자열이 pool에 있다면 이를 불러오는 방식  
                - cf) 싱글톤 패턴(Singletone pattern)과의 비교
                    - 공통점  
                    객체 하나만 생성하고 활용하는 패턴이다.  
                    - 차이점
                        - 플  
                            - 특정 옵션이 다르면 새로운 객체가 생성이 됨(여러 종류가 하나씩 존재)
                            - ex) 나무 객체가 빨,파,주황색 별로 있는 것
                        - 싱  
                            - 특정 클래스에 단 하나의 객체만을 생성할 수 있음  
                            (하나의 클래스에 단 하나의 인스턴스를 생성하고 변수를 필요 시 변경해가며 씀)
                            - ex) 하나의 나무 객체의 색깔을 바꿔야함

                
        - `반환 타입의 하위 타입 객체를 반환할 수 있음`  
            반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성’을 제공함  
                - 효과  
                    API를 만들 때 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어
                    API를 작게 유지할 수 있다.  
                    => 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는
                        인터페이스 기반 프레임워크(아이템 20)를 만드는 핵심 기술이됨.  
                    => API가 작아진 것은 물론 개념적인 무게，즉 프로그래머가 뼈를 사용하기 위해 익혀야 하는 개념의 수와 난이도도 낮췄다는 것이다. 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 된다. 나아가 정적 팩터리 메서드를 시용하는 클라이 언트는 얻은 객체를 (그 구현 클래스가 아닌) 인터페이스만으로 다루게 된다(아이템 64) 물론 이는 일반적으로 좋은습관이다.  
                - 자바 8 전에는 인터페이스에 정적 메서드를 선언할 수 없었다.  
                그렇기 때문에 이름이 “Type" 인 인터페이스를 반환하는 정적 메서드가 필요하면  
                “Types" 라는 (인스턴스화 불가인) 동반 클래스(companion class)를 만들어서  
                그 안에 정의하는 것이 관례였다.
                (EX - 단 하나의 인스턴스화 불가 클래스인 java.util.Collections 에서 정적 팩터리 메서드를 통해서만 자바 컬렉션 프레임워크를 얻도록 )  
            자바 8부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸기 때문에 인스턴스화 불가 동반 클래스를 둘 이유가 별로 없다.  
            동반 클래스에 두었던 public 정적 멤버들 상당수를 그냥 인터페이스 자체에 두면 되는 것이다.  
            하지만 정적 메서드들을 구현하기 위한 코드 중 많은 부분은 여전히 별도의 package-private 클래스에 두어야 할 수 있다.  
            자바 8 에서도 인터페이스에는 public 정적 멤버만 허용하기 때문이다.  
            자바 9에서는 private 정적 메서드까지 허락하지만 정적 필드와 정적 멤버 클래스는 여전히 public이어야 한다.
        - `입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있음`  
            반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.(심지어 다음 릴리스에서는 또 다른 클래스의 객체여도)
            - ex)
                EnumSet 클래스(아이댐 36)는 public 생성자 없이 오직 정적 팩터리만 제공하는데 Open]DK에서는 원소의 수에 따라 두 가지 하위 클래스 중 하나의
인스턴스를 반환한다.  
만약 원소가 적을 때 RegularEnumSet을 사용할 이점이 없어진다면 다음 릴리스 때는 이를 삭제해도 아무 문제가 없다.  
비슷하게 성능을 더 개선한 세 번째 , 네 번째 클 래스를 다음 릴리스에 추가할 수도 있다.  
하지만 클라이언트는 팩터리가 건네 주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다 EnumSet의 하위클래스이기만 하면 되는 것이다.  
        - `정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.`  
        서비스 제공자 프레임워크(service provider framework)를 만드는 근간이 된다.  
            - 서비스 제공자 프레임워크
                - 제공자(provider) == 서비스의 구현체
                    이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여 클라이언트를 구현체로부터 분리해준다.  
                - ex - JDBC(Java Database Connectivity)  
                    대표적인 서비스 제공자 프레임워크  
                - 핵심 컴포넌트
                    - 서비스 인터페이스(service interface)  
                        구현체의 동작을 정의  
                        ex) JDBC/Connection
                    - 제공자 등록 API(provider registration API)  
                        제공자가 구현체를 등록할 때 사용  
                        ex) Drive rManager.registerDriver
                    - 서비스 접근 API(service access API)  
                        클라이언트가 서비스의 인스턴스를 얻을 때 사용  
                        클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조
        건을 명시할 수 있는데 생략하면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다.  
         => 서비스 제공자 프레임워크의 근간이라고 한‘유연한정적 팩터리’의 실체  
                        ex) DriverManager.getConnection
                    - 서비스 제공자 인터페이스(service provider interface)  
                        서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명.
                        이게 아니면 각 구현체를 인스턴스로 만들 때 리플렉션(아이템
        65)을 사용해야 한다.  
            ex) Driver
            
        

- 정적 팩터리 메서드의 단점
    - 요약
        - 정적 팩터리 메서드만으로는 하위 클래스를 만들 수 없다.  
            ( ∵ 상속을 하려면 public/protected 생성자가 필요 )
        - 정적 팩터리 메서드는 프로그래머가 찾기 어려움

    - 상세
        - 정적 팩터리 메서드만으로는 하위 클래스를 만들 수 없다.   
            ( ∵ 상속을 하려면 public/protected 생성자가 필요 )  
            컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다는 것.    
상속보다 컴포지션을 사용(아이템 18)하도록 유도하고 불변 타입(아이댐 17)으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.    
        - 정적 팩터리 메서드는 프로그래머가 찾기 어려움  
        생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.  
        자바독이 알아서 해주는 날 전까지는 API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약을 따라 짓는 식으로 문제를 완화해줘야 한다.  
            - 정적 팩터리 메서드에 흔히 사용하는 명명 방식들
                - from  
                매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드  
    ex) ```java Date d Date.from(instant);```
                - of  
                여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드  
                ex) ```java Set<Rank> faceCards EnumSet.of(JACK, QUEEN, KING);```
                - valueOf  
                from과 of 의 더 자세한 버전  
                ex) ```java Biglnteger prime = Biglnteger.valueOf(Integer.MAX_VALUE);```
                - instance 혹은 getInstance  
                (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임을 보장하지는 않는다.  
                ex) ```java StackWalker luke StackWalker.getlnstance(options);```
                - create 혹은 newlnstance  
                instance 혹은 getInstance와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장한다.  
                ex) ```java Object newArray Array.newlnstance(classObject, arrayLen);```
                - getType  
                getInstance와 같으나 생성 할 클래스가 아닌 다른 클래스에 팩터리 메서 드를 정의할 때 쓴다.  
                (“Type"==팩터리 메서 드가 반환할 객체의 타입)  
                ex) ```java FileStore fs = Files.getFileStore(path)```
                - newType
                newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터  
                리 메서드를 정의할 때 쓴다.(“Type"==팩터리메서드가 반환할 객체의 타입)  
                ex) ```java BufferedReader br = Files.newBufferedReader(path);```
                - type  
                getType과 newType의 간결한 버전  
                ex) ```java List<Complaint> litany = Collections.list(legacyLitany);```
    
- 요약  
    두 가지 다 상대적인 장단점이 존재하지만  
    정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로  
    무작정 public 생성자를 제공하던 습관은 고치자.

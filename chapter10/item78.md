#  아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

<br>

## synchronized(동기화)의 기능
### `배타적 수행 - 해당 메서드나 블록을 한 번에 한 스레드씩 수행하도록 보장한다.`  
    - 객체 접근 과정  
    한 객체가 일관된 상태를 가진 채 생성됨(아이템 17)  
    -> 이 객체에 접근하는 메서드는 그 객체에 락(lock)을 건다.  
    -> 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정해서 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.  
    => 동기화를 제대로 사용하면 이 객체의 상태가 늘 일관되는 것을 볼 수 있다.(일관성이 깨진 것을 볼 수 없음)

### `스레드 간의 통신 - 한 스레드가 만든 변화를 다른 스레드에서도 확인할 수 있게 해준다.`   
(동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.)  
동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게해준다.  
=> 스레드 사이의 안정적인 통신에 꼭 필요하다.  
(이유 - 자바의 메모리 모델에 한 스레드가 만든 변화가 다른 스레드에게 언제, 어떻게 보이는지가 규정되어있음 )

> 원자적 데이터와 동기화
- long, double 외의 변수를 읽고 쓰는 것 - 원자적(atomic)  
여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도  
항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻

자바 언어 명세는 스레드가 필드를 읽을 때 항상 ‘수정이 완전히 반영된’ 값을 얻는다고 보장하지만  
한 스레드가 저장한 값이 다른 스레드에게 ‘보이는가’는 보장하지 않는다.  
성능을 위해서 원자적 데이터를 읽고 쓸 때는 동기화를 하지 않는 건 어리석다.  
(이유 - 공유중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하게 되면 리스크가 큼)0
- ex) 다른 스레드를 멈추는 방법  
    - 잘못된 방법 - Thread.stop()  
    안전하지 않아 이미 오래전에 사용 자제(deprecated) API로 지정되었다.  
    이 메서드를 사용하면 데이터가 훼손될 수 있으므로 Thread.stop은 사용하지 말자.
    - 올바른 방법  
    스레드는 자신의 boolean 필드의 값을 false로 초기화해놓고  
    다른 스레드에서 이 스레드를 멈추고자 할 때, 폴링하면서 true로 변경해서 멈추게 한다.  
    boolean 필드를 읽고 쓰는 작엽은 원자적이라 어떤 프로그래머는 이런 필드에 접근할 때 동기화를 제거하기도한다.


## 예제
- 동기화하지 않아서 발생한 문제점
    - 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤 보게 될지 보증할 수 없다.
    - 현상
        가상머신이 최적화 기법인 끌어올리기(호이스팅. hoisting)을 수행한다.
        -> 프로그램이 응답 불가 상태(liveness failure)가 돼서 진전이 없음
- 대안
    stopRequested 필드의 동기화 처리

- 읽고 쓰기 메서드를 대상으로 둘 다 동기화가 되어야 한다.
- 통신 목적으로만 사욛ㅇ되면 동기화 없이도 운자적으로 동작한다.

### volatile 한정자 키워드
- 반복문에서 매번 동기화 비용이 크진 않지만 속도가 더 빠른 대안
- 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장해준다.
- 주의점
    - 안전실패(safety failure) 유발 가능
    프로그램이 잘못된 결과를 계산해내는 오류
        - 대안
            1. synchronized 한정자 붙이기
            2. volatile 제거
            3. int 대신 long을 사용하거나 예외처리 하기
            4. java.util.concurrent.atomic 패키지의 AtomicLong 사용하기

## 특징
- 불변 데이터만 공유하거나 가변 데이터는 단일 스레드엣만 쓰도록 하는 것이 좋다.
- 유지보수 과정에서도 이러한 정책을 지켜지도록 문ㅅ화 하자
- 사용하려는 프레임워크와 라이브러리를 깊이 이해하자.

<br>

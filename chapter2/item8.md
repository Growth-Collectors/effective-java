# [Item 8] finalizer와 cleaner 사용을 피하라

finalize 메소드란?
- 자바의 모든 클래스는 최상위 클래스인 Object 클래스의 여러 메서드를 포함하고 있음
- 객체 소멸자라고 말하는 finalize 메서드도 그 메서드들 중 하나
- 존재이유가 불분명한 메서드
  - 자바에서 메모리는 개발자가 직접 관리하지 않는다. 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터 가 담당하고, 프로그래머 에게는 아무런 작업도 요구하지 않는다. 
- "종료자는 사용하면 안 된다. 예측이 불가능하고 대체로 위험하고 일반적으로 필요하지 않다."


권장: GC를 이용해서 메모리 회수하기 
```
ObjectEx obj4; // ObjectEx 객체와 obj4 레퍼런스
obj4 = new ObjectEx(); // ObjectEx()클래스에 obj4레퍼런스 생성
obj4 = new ObjectEx(); // ObjectEx()클래스에 obj4레퍼런스 재 생성, 따라서 이전 관계는 끊어짐

// 언젠가 GC가 적절한 타이밈에 메모리를 회수해 줌
```

비권장: 소멸자를 호출해서 자원 회수하기 
```
@Override
	protected void finalize() throws Throwable {
		System.out.println(" -- finalize() method --");
		super.finalize();
  }
```

`finalizer` 사용이 불러온 재앙
- `finalizer` 처리 스레드는 애플리케이션 스레드보다 우선 순위가 낮다는 문제가 있음
  - 원인을 알 수 없는 OutOfMemoryError를 내며 죽는 GUI 애플리케이션을 디버깅해 보니 그래픽스 객체 수천 개가 `finalizer` 대기 열에서 회수되기만을 기다 라고 있었다. 
  불행히도 `finalizer` 스레드는 다른 애플라케이션 스레드보다 우선 순위가 낮아서 실행될 기회를 제대로 얻지 못한 것이다. 
- 그래서 JAVA9 에서부터는 `cleaner` 로 대체
  - `cleaner` 에서는 자신을 수행할 스레드를 제어할 수 있다
  - 그러나 여전히 수행 시점은 보장하지 않음

`finalizer`는 오히려 GC의 자원회수를 방해한다
- 저자가 생성한 AutoCloseable 객체
  - 가비지 컬렉터가 수거하기까지 12ns
  - finalizer를 사용하면 550ns
  - 안전망 방식의 `finalizer`에서는 660s (아래에서 설명..그치만 어쨋든간 느리다..) 
- finalizer가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다.

`finalizer`를 사용한 보안 공격  
- 객체가 소멸된 후에도 객체에 접근가능, 소멸된 객체의 인스턴스도 실행가능
- 그치만 GC가 언젠간 객체를 정리해 주지 않냐? 그 뒤로는 안전하겠지? No
  - 생존 트릭: 다음과 같이 finalize안에 자기참조를 넣으면 "참조"효과로 GC에게 정리당하지도 않고 프로그램 종료될 때까지 좀비상태로 남아있게 됨. 
  - ```
    public class Zombie {
    static Zombie zombie;

    public void finalize() {
      zombie = this;
      }
    }
    ```
finalizer attack 방어하기
- `finzlize` 상속 금지
  - 위와같은 코드를 짜는 개발자는 없다. subclassing 과정에서 실수로 저런 코드가 만들어지기 때문
  - 	final protected void finalize() {
- 클래스 상속 금지 
  - 위의 방법보다 조금 더 강한 제약
  -  	public final class AccountOperations {
- (workaround)Boolean flag를 사용해라 
  - 공격자가 임의 메서드 접근을 성공했다 하더라도, 메서드 안에 검증루틴을 하나 더 넣어서 실행은 안되도록 막기
  -  ```
     private boolean isUserAuthorized = false;
     public void transferMoney(double amount) {
       if (!isUserAuthorized) {
         System.out.println("You are not authorized");
         return;
       }
       System.out.println("Transferring " + amount + " to beneficiary");
     }
     ```
이쯤이면 cleaner와 finalizer는 대체 어디에 쓰는 물건일까? 
- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할(늦더라도 꼭 회수해라~ 를 명시적으로 선언)
- 네이티브 피어(native peer)와 연결된 객체에
- 네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체 
- 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지못한다.

안전망으로써의 cleaner 사용
- clean 호출될 때 행동을 내가 지정해주고 싶다면 사용 
```
    //cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles) {
    	state = new state(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override public void close() {
    	cleanable.clean();
    }    
```
- 하지만 이것역시 반드시 실행되리라는 보장 없음 
- 클라이언트가 모든 Room 생성을 try-with-resources 블록으로 감썼다변 자동 청소는 전혀 필요하지 않다. 

참고: https://www.zehye.kr/java/2019/08/22/11java_constructor/

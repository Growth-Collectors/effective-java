# 아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라

- wait와 notify는 올바르게 사용하기 까다로우니 고수준 동시성 유틸리티를 사용한다.

## java.util.concurrent의 고수준 유틸리티
### 세 범주
- 실행자 프레임워크
- 동시성 컬렉션(concurrent collection)
- 동기화 장치(synchronizer)
 
### 동시성 컬렉션(concurrent collection)
- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다.
- 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.
- 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.
    - 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 것은 불가능하다.
    - 그리하여 여러 기분 동작을 하나의 원자적 동작으로 묶는 ‘상태 의존적 수정’ 메서드들이 추가되었다.
    - 이 메서드들은 아주 유용해서 자바 8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드 형태로 추가되었다.

- ConcurrentMap으로 구현한 동시성 정규화 Map (최적은 아니다)
```java
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();

public static String intern(String s) {
    String previousValue = map.putIfAbsent(s, s);
    return previousValue == null ? s : previousValue;
}
```
- Map의 putIfAbsent(key, value) 메서드는 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 집어 넣는데, 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다.
- String.intern의 동작을 흉내낸 메서드이다.
 
- ConcurrentMap으로 구현한 동시성 정규화 Map (위 코드보다 더 빠름)
```java
public static String intern(String s) {
    String result = map.get(s);
    if (result == null) {
        result = map.putIfAbsent(s, s);
        if (result == null)
            result = s;
    }
    return result;
}
```
- ConcurrentHashMap은 get 같은 검색 기능에 최적화되었다.
- 따라서, get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하면 더 빠르다.
- 이 메서드는 String.intern 보다 6배 빠르다. (다만, String.intern에는 누수 방지 기능도 있다.)
- 대표적인 예로, 이제는 Collections.synchroizedMap 보다는 ConcurrentHashMap을 사용하는 게 훨씬 좋다.
- 동기화된 Map을 동시성 Map으로 교체하는 것만으로 동시성 애플리케이션의 성능은 극적으로 개선된다.
 
### BlockingQueue
- 컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.
    - 예로, Queue를 확장한 BlockingQueue에 추가된 메서드 중 take는 큐의 첫 원소를 꺼낸다.
    - 이 때, 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다.
    - 이런 특성 덕에 BlockingQueue는 작업 큐(생산자-소비자 큐)로 쓰기 적합하다.
- 작업 큐는 하나 이상의 생산자(producer) 스레드가 작업(work)을 큐에 추가하고, 하나 이상의 소비자(consumer) 스레드가 큐에 있는 작업을 꺼내 처리하는 형태다.
- ThreadPoolExecutor를 포함한 대부분의 실행자 서비스(아이템 80) 구현체에서 이 BlockingQueue를 사용한다.
 
### 동기화 장치
- 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.
- 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다.
- CyclicBarrier와 Exchanger는 그보다 덜 쓰이며, 가장 강력한 동기화 장치는 바로 Phaser다.
 
### CountDownLatch
- 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.
- CountDownLatch의 유일한 생성자는 int 값을 받으며, 이 값이 래치의 countDown 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.
- 이 장치를 활용하여 유용한 기능을 쉽게 구현할 수 있다.
- 예를 들어, 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크를 구축해보자.
- 이 프레임워크는 메서드 하나로 구성되며, 이 메서드는 동작들을 실행할 실행자와 동작을 몇 개나 동시에 수행할 수 있는지를 뜻하는 동시성 수준(concurrency)을 매개변수로 받는다.
    - 타이머 스레드가 시계를 시작하기 전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다.
    - 마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 '시작 방아쇠'를 당겨 작업자 스레드들이 일을 시작하게 한다.
    - 마지막 작업자 스레드가 동작을 마치자마자 타이머 스레드는 시계를 멈춘다.
    - 위의 기능을 wait와 notify만으로 구현하려면 아주 난해하고 지저분한 코드가 탄생하지만, CountDownLatch를 쓰면 아래처럼 직관적으로 구현할 수 있다.

```java
public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done  = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
        executor.execute(() -> {			
            ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
            try {
								
                start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                action.run();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                done.countDown(); // 타이머에게 작업을 마쳤음을 알린다.
            }
        });
    }

    ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
}
```
- 이 코드는 카운트다운 래치를 3개 사용한다.
- ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다.
    - 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다.
    - 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다.
- 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다.done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다.
- 타이머 스레드는 done 래치가 열리자마자 깨어나 종료 시각을 기록한다.

#### 추가 정보
- time 메서드에 넘겨진 실행자(executor)는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 결코 끝나지 않을 것이다
    - 이런 상태를 스레드 기아 교착상태(thread starvation deadlock)라 한다.
- InterruptedException을 캐치한 작업자 스레드는 Thread.currentThread().interrupt() 관용구를 사용해 인터럽트(interrupt)를 되살리고 자신은 run 메서드에서 빠져나온다.
    - 이렇게 해야 실행자가 인터럽트를 적절하게 처리할 수 있다.(무슨 말인지 이해 못함)
- **시간 간격을 잴 때는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용**하자.    
    - System.nanoTime은 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.
    - 이 예제의 코드는 작업에 충분한 시간(예: 1초 이상)이 걸리지 않는다면 정확한 시간을 측정할 수 없을 것이다. 
    - 정밀한 시간 측정은 매우 어려운 작업이라, 꼭 해야 한다면 jmh 같은 특수 프레임워크를 사용해야 한다.
 
## wait와 notify
- 새로운 코드라면 언제나 wait와 notify가 아닌 동시성 유틸리티를 써야 한다.
- 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있다.
### wait
- wait 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.
- 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.
- wait를 사용하는 표준 방식은 다음과 같다.
```java
synchronized (obj) {
    while (<조건이 충족되지 않았다>)
        obj.wait(); // (락을 놓고, 깨어나면 다시 잡는다.)
        
    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```
- wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라.
- 반복문 밖에서는 절대로 호출하지 말자.
- 이 반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다.
- 대기 전에 조건을 검사하여 조건이 이미 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.
- 만약 조건이 이미 충족되었는데 스레드가 notify(혹은 notifyAll) 메서드를 먼저 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.
- 대기 후에 조건을 검사하여 조건이 충족되지 않았다면 다시 대기하게 하는 것은 안전 실패를 막는 조치다.
- 만약 조건이 충족되지 않았는데 스레드가 동작을 이어가면 락이 보호하는 불변식을 깨뜨릴 위험이 있다.

### 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황
- 스레드가 notify를 호출한 다음 대기 중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
- 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 
    - 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다.
    - 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
- 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
- 대기 중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 있다.허위 각성(spurious wakeup)이라는 현상이다.
 
### notify vs notifyAll
- 일반적으로 언제나 notifyAll을 사용하라는 게 합리적이고 안전한 조언이 될 것이다.
- 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다.
- 다른 스레드까지 깨어날 수도 있긴 하지만, 그것이 프로그램의 정확성에는 영향을 주지 않을 것이다.
- 깨어난 스레드들은 기다리던 조건이 충족되었는지 확인하여, 충족되지 않았다면 다시 대기할 것이다.
- 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화할 수 있다.

### notifyAll
- 하지만 이상의 전제조건들이 만족될지라도 notify 대신 notifyAll을 사용해야 하는 이유가 있다.
- 외부로 공개된 객체에 대해 실수로 혹은 악의적으 로 notify를 호출하는 상황에 대비하기 위해 wait를 반복문 안에서 호출했 듯, notify 대신 notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait를 호출하는 공격으로부터 보호할 수 있다.
- 그런 스레드가 중요한 notify를 삼켜버린다면 꼭 깨어났어야 할 스레드들이 영원히 대기하게 될 수 있다.
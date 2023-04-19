# 아이템 79. 과도한 동기화는 피하라

- 충분하지 못한 동기화도 문제이지만 과도한 동기화도 문제다. 
    - 성능을 떨어뜨리고, 교착상태(Deadlock)에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.
- **응답 불가**와 **안전 실패**를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.
    - 즉, 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체(아이템 24)를 호출해서도 안 된다.

## 외계인 메서드(alien method)
- 그 메서드가 무슨 일을 할지 알지 못하며 통제할 수도 없음을 뜻한다.
- 외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

### 예시
- 아래는 어떤 집합(Set)을 감싼 래퍼 클래스이고, 집합에 원소가 추가되면 알림을 받을 수 있는 관찰자 패턴을 사용한 예제이다.
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element); // 외계인 메서드 호출
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element); // notifyElementAdded 호출
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element); // notifyElementAdded 호출
        return result;
    }
}
```
- 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다.

```java
@FunctionalInterface
public interface SetObserver<E> {
    // ObservableSet에 원소가 추가되면 호출된다. - 콜백용 인터페이스
    void added(ObservableSet<E> set, E element);
}
```
- 위 함수형 인터페이스는 BiConsumer<ObservableSet\<E>, E> 와 구조적으로 같다

### 정상 동작
```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    
    set.addObserver((s,e) -> System.out.println(e));
    
    for (int i = 0; i < 100; i++)]
        set.add(i);
}
```
- 0부터 99까지를 출력한다.

### 동기화 블록 안에서 외계인 메서드를 호출 - 예외 발생
- 위에서 람다를 활용한 것과 달리 익명 클래스를 사용했다. 
    - 해당 로직을 위해 함수 객체 자신을 넘겨야 하는데, 람다는 자기 자신을 참조할 수단이 없다.
```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23)
                s.removeObserver(this); // remove 메서드를 호출
        }
    });

    for (int i = 0; i < 100; i++)
        set.add(i;)
}
```
- 0부터 23까지 출력한 후 ConcurrentModificationException가 발생한다. 
    - 관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 Observer들의 리스트를 순회하는 도중이기 때문이다.
- notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있어 동시 수정이 일어나지 않도록 보장하나, 정작 자신이 콜백을 거쳐 되돌아와 수정하려고 시도하는 것을 막지 못한다.
    - added 메서드에서 ObservableSet.removeObserver 메서드를 호출하고, 또 여기서 observers.remove 메서드를 호출한다. 
        - 순회하고 있는 리스트에서 원소를 제거하려고 하는 것이다. 즉, 허용되지 않은 동작이다.


### 쓸데없는 백그라운드 스레드를 사용하는 관찰자
- 실행자 서비스(ExecutorService)를 사용하여 다른 스레드가 Observer를 구독 해지하도록 한다.
```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());

    // 메인 스레드가 관찰자에 대한 락을 얻음
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if (e == 23) {
                ExecutorService exec = Executors.newSingleThreadExecutor();
                try {
                    // lock 을 얻으려고 하지만, 데드락 발생
                    // 특정 태스크가 완료되기를 기다린다. - submit의 get 메서드
                    exec.submit(() -> s.removeObserver(this)).get();
                } catch (ExecutionException | InterruptedException ex) {
                    throw new AssertionError(ex);
                } finally {
                    exec.shutdown();
                }
            }
        }
    })

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
- 위 코드는 예외는 발생하지 않지만 **교착상태(Deadlock)**에 빠진다. 
    - 백그라운드 스레드가 s.removeObserver 메서드를 호출하면 Observer를 잠그려 시도하지만 락을 얻을 수 없다. 
    - 메인 스레드가 이미 락을 잡고 있기 때문이다.
    - 동시에 메인 스레드는 관찰자를 제거하기만을 기다린다.
- 억지스러운 예지만 어쨌든 이럴 수도 있다~
    - GUI 툴킷과 같은 시스템에서  동기화된 영역 안에서 외계인 메서드를 호출해 교착상태에 빠지는 사례가 자주 있다.

### 불변식이 깨지는 경우
- 위의 예는 운이 좋은 케이스다. 여기에 더해 불변식이 임시로 깨지는 경우도 존재한다.
- 이런 경우에 자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다.
- 첫번째 예의 경우, 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공할 것이다.
    - 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행 중임에도 불구하고
- **재진입 가능 락**
    - 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)으로 변모시킬 수 있다.

### 예외/교착상태 해결방법
- **외계인 메서드 호출을 동기화 블록 바깥으로 옮기면** 된다.
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```
- 이러한 동기화 영역 바깥에서 호출되는 외계인 메서드를 **열린 호출(open call)**이라고 한다.
    - 외계인 메서드의 실행 소요 시간이 어느정도일지 예측할 수 없는 상황에서 동기화 영역 안에서 호출된다면, 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야 하는 문제가 있다.
    - 즉, 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선한다.

- 조금 더 나은 방법으로는 자바의 동시성 컬렉션 라이브러리를 사용한다. 
    - 스레드 안전하고 관찰 가능
    - 명시적인 동기화 영역이 사라진다.
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```
- CopyOnWriteArrayList는 ArrayList를 구현한 클래스
    - 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되었다.
    - 내부의 배열은 절대 수정되지 않아 순회할 때 락이 필요 없어 매우 빠르다. 
    - 다른 용도로 사용된다면 매번 복사해서 느리지만, 수정할 일이 적고 순회만 빈번하게 일어난다면 Observer 리스트 용도로는 최적이다.

### 기본 규칙
- 동기화 영역에서는 가능한 한 일을 적게 한다.
- 락을 얻고, 공유 데이터를 검사하고, 필요하면 수정하고 락을 놓는다.

### 성능 측면
- 과도한 동기화는 경쟁으로 인해 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다.
- 또한 JVM의 코드 최적화를 제한하는 점도 고려한다.

### 가변 클래스를 작성시 두 가지 선택
- 첫 번째는 동기화를 하지 않고, 그 클래스를 사용해야 하는 클래스가 외부에서 동기화하는 것이다. 
    - java.util 라이브러리
- 두 번째는 동기화를 내부에서 수행해 thread-safe 한 클래스로 만드는 것이다. 
    - 다만 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 개선할 수 있을 때 선택해야 한다. - java.util.concurrent 라이브러리(아이템 81)
- 합당한 이유가 있을 때만 내부에서 동기화를 수행하고, 동기화했는지 여부를 문서에 명확히 밝히자.

#### 클래스 내부 동기화
- 락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원해 동시성을 높일 수 있다.
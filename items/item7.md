# [Item 7] 다 쓴 객체 참조를 해제하라
- _"C, C++처럼 메모리를 직접 관리해야 하는 언어를 쓰다가 자바처럼 가비지 컬렉터를갖춘 언어로 넘어오면 프로그래머의 삶이 훨씬 평안해진다. 다 쓴 객체 를 알아서 회수해가니 말이다. 처음 경험할 때는 마법을 보는 듯했다."_

- JVM의 가비지 컬렉션
  - 불필요한 Heap 메모리를 청하는 것
  - Garbage = 불필요한 오브젝트 = null 처리 되었거나 해당 인스턴스를 참조하는 부분이 없는 오브젝트
  - JVM: 그 누구도 기억하지 못하는 오브젝트는 사라져 마땅하다
  - 하지만 JVM은 충분히 똑똑하지 못하다는 점...
- 아래 코드에서의 문제는?
```Java
public class Stack {
  
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
  
    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }
  
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return this.elements[--size]; // memory leak point. 개발자 입장에선 자명히 보이는데! 
    }
  
    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
어떻게 해결할까? 해당 참조를 다 썼을 때 null 처리(참조 해제)할 것
- 이 객체가 여전히 참조된다면 JVM입장에서는 객체가 쓰레기인지 아닌지 알 길이 없다
- 개발자가 직접 null처리 해주자
```Java
public Object pop() {
  if (size == 0) {
    throw new EmptyStackException();
  }
  Object value = this.elements[--size];
  this.elements[size] = null;
  return value;
}
```
참고로 객체의 유효 범위를 지정해 주는것도 메모리가 자연스레 정리되기 때문에 권장함 
```Java
for (Element e c) {
... // e로 무언가를 한다.
}
```
캐시/콜백 역시 메모리 누수를 일으키는 주범이다 
- 어딘가에 메모리주소를 보관하는 구조이기 때문에 
- 캐시는 key:value 쌍이다
- ```
  (바라보는 구조) 우리의코드 -> key -> value 
              -----------------         // 1) 이 링크가 사라지더라도 (=쓸모없는 캐시)
                        --------------- // 2) 이 링크가 여전히 존재 -> 가비지 콜렉션이 일어나지 않는다 
따라서 캐시를 관리할 때에는 WeakHashMap 을 쓰도록 하자
  - WeakHashMap
    - Key에 대한 참조가 더 이상 존재하지 않게 되면, Value를 가져올 수 있는 방법이 없다고 판단하여, 해당 Key-Value 쌍은 자동으로 삭제되는 Map이다.
  - 원리? 레퍼런스를 WeakReference로 선언하면 JVM이 자세히 검사해준다 (2)를 청소할때 `1)`도 검사해준다)


# 29. 제네릭 타입으로 만들어라

## Generic 타입을 써야 하는 이유
- 클라이언트 코드(main 메서드)에서 직접 형변환하는 것은 휴먼에러가 날 수 있다
- 새로운 타입 설계 시에는 꼭 형변환 없이도 사용할 수 있게 하자
- 기존 타입에서 Generic이어야 하는 개체가 있다면 변경하자
- 클라이언트 코드에 영향을 안 주면서, 새로운 사용자에게는 편리함을 줄 수 있기 때문

## Stack 리팩터링
### 클래스 선언에 타입 매개변수 추가하자 (?)
Object 배열을 활용한다면
```
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
 
   // Corrected version of pop method (Page 27)
   public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
   }
}
```

이런 식으로 꺼낼 때 Object 타입이어야 하므로 런타임 에러가 날 가능성이 크다

모든 Object를 Generic으로 바꾸기만 한다면?


```
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```
Generic 매개변수 타입으로 배열을 만들 수는 없다!
어떻게 해결해야 할까? 





### 해결책 1. Object 배열 생성->Generic 배열로 캐스팅

```
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack() {
        elements =(E[]) new E[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

- 제네릭 배열을 선언하고, 생성자에서 Object 배열 만들기 -> E[]로 형변환
- 만들어두면 원하던 타입대로 꺼낼 수 있음
- 런타임 경고를 서프레스워닝으로 무시할 것
- 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다.
- 원소의 타입은 항상 E
- But heap 오염 발생 

두가지 방식

배열을 어딘가에서 리턴해주면  변경할 수 있는데
들어오는 타입은 E임

형변환을 딱 한 번만 하면 됨

가독성도 좋지만 힙 오염이 발생할 수 있음
힙 오염 = 
GenericArrayError
E타입 배열 말고
오브젝트 배열을 쓰고
제네릭 타입으로만 푸시받기
### 해결책 2.Object로 생성하, Generic 타입으로만 pop

```
public class Stack<E> {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack() {
        elements =(E[]) new E[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }
 
    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = (E) elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```


첫 번째 방식에서는 형변환을 배열 생성 시 단 한번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 



# 아이템 58. 전통적인 for 문보다는 for-each 문을 사용하라

### for-each  = 향상된 for 문

```java
for (Element e : elements) {
 ... // e로 무언가를 한다.
}
```

- 콜론 ( : ) 은 “안의 ( in )” 라고 읽는다.
- `elements 안의 각 원소의 e에 대해` 라고 읽는다.

### for-each를 사용해야 하는 이유?

- 코드의 가독성과 유지 보수성이 향상되고 버그 발생 가능성이 줄어든다.
- 인덱스 변수를 사용하지 않기 때문에 코드 작성 시 실수할 가능성이 줄어든다.
- 전통적인 for문과 속도가 똑같다. 반복 대상이 컬렉션이든 배열이든 상관없다.

### for-each를 사용하지 못하는 경우는?

- **파괴적인 필터링(destructive filtering)**
  컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
    - 리스트나 배열을 인-플레이스로 필터링하여 원래 컬렉션의 상태를 변경하는 것을 말한다. 이 경우 for-each 루프를 사용할 수 없다. 왜냐하면 내부적으로 반복자(iterator)를 사용하며, 기본 컬렉션에서 요소를 제거할 수 있는 옵션을 제공하지 않기 때문.

    <aside>
    💡 **여기서 말하는 인-플레이스란?**
    데이터를 다루는 알고리즘에서, 입력 데이터를 직접 수정하거나 추가적인 메모리를 사용하지 않고, 입력 데이터 내에서 계산을 수행하는 방식을 말한다.

  파괴적 필터링에서 인-플레이스는 원래의 리스트나 배열을 수정하면서 필터링하는 것을 의미한다. 이렇게 하면 원래의 데이터를 변경하므로, 추가 메모리를 사용하지 않아도 되고, 작업을 더 빠르게 수행할 수 있다.

  하지만 for-each 루프를 사용하여 인-플레이스 필터링을 하려고 하면, 루프 내부에서 컬렉션을 수정하면서 반복자(iterator)와 충돌이 발생하여 ConcurrentModificationException 예외가 발생할 수 있다.
  따라서 인-플레이스 필터링을 할 때에는 명시적으로 반복자를 사용하여 요소를 제거하는 것이 좋다.

    </aside>

    ```java
    List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
    
    for (Integer number : numbers) {
        if (number % 2 == 0) {
            numbers.remove(number);
        }
    }
    ```

- **변형(transforming)**
  리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
    - 리스트나 배열을 요소를 수정하여 새로운 리스트나 배열을 만드는 것.
    - 이 경우 for-each 루프를 사용할 수는 있지만, 새로운 리스트나 배열을 만드는 과정에서 일시적으로 두 개의 리스트나 배열을 가지고 있어야 하므로 메모리 사용량이 더 많아질 수 있다.

        ```java
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        List<Integer> doubledNumbers = new ArrayList<>();
        
        for (Integer number : numbers) {
            doubledNumbers.add(number * 2);
        }
        ```

- **병렬 반복(parallel iteration)**
  여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 한다.
    - 병렬 스트림(parallel stream)을 사용하여 요소를 병렬로 처리해야한다.

        ```java
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        numbers.parallelStream().forEach(System.out::println);
        ```
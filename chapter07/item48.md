# [Item 48] 스트림 병렬화는 주의해서 적용하라
         "주류 언어중, 동시성 프로그래밍 측면에서 자바는 항상 앞서갔다. wait/notify -> Executor -> fork-join -> parallel 스트림의 순서로 자바로 동시성 프로그램을 작성하기가 점점 쉬워지고는 있으나, 이를 올바르고 빠르게 작성하는 일은 여전히 어려운 작업이다. 동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다."


- 아이템 45에서 다루었던 메르센 소수를 생성하는 프로그램을 다시 살펴보자.
```java
public static void main(String[] args) {
   primes().map(p -> TWO.pos(p.intValueExact()).subtract(ONE))
      .filter(mersenne -> mersenne.isProbablePrime(50))
      .limit(20)
      .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
   return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```
- 이 프로그램을 내 컴퓨터에서 실행하면 즉가가 소수를 찍기 시작해서 12.5초 만에 완료된다. 속도를 높이고 싶어 스트림 파이프라인의 parallel()을 호출하겠다는 순진한 생각을 했다고 치자. 성능은 어떻게 변할까? 
- 안타깝게도 이 프로그램은 아무것도 출력하지 못하면서 CPU는 90%나 잡아먹는 상태가 무한히 계속된다. 
- 환경이 아무리 좋더라도 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다. 그런데 위의 코드는 두 문제를 모두 지니고 있다. 
- 파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다. 그런데 이 코드의 경우 새롭게 메르센 소수를 찾을 때마다 그 전 소수를 찾을 때보다 두 배 정도 더 오래 걸린다. 
- 즉, 원소 하나를 계산하는 비용이 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다는 뜻이다. 

    - 스트림 파이프라인을 마구잡이로 병렬화하면 안 된다. 성능이 오히려 끔찍하게 나빠질 수도 있다.


- 대체로 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다.

   - 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다. 
   - 또 다른 중요한 공통점은 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다는 것이다. 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.
   - 참조 지역성은 병렬화할 때 아주 중요한 요소로 작용한다. 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다. 


- 스트림 파이프라인의 종단연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다. 
- 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한될 수밖에 없다. 종단 연산 중 병렬화에 가장 적합한 것은 축소다. reduce, min, max, count, sum가 속한다. 
- anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 적합하다. 
- 반면, 가변 축소를 수행하는 collect 메서드는 병렬화에 적합하지 않다. 합치는 부담이 너무 크다.


- 직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리게 하고 싶다면 spliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 성능을 강도 높게 테스트하라. 아쉽지만 이 책에서는 다루지 않는다. 
- 스트림을 잘못 병렬화하면 (응답 불가능 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다

- 결과가 잘못되거나 오동작하는 것은 안전실패 (Safety failure)라고 하는데, 프로그래머가 제공한 함수가 명세대로 동작하지 않을 때 벌어질 수 있다. 
- Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다.    
  - "Stream의 reduce 연산에 건네지는 accumulator, combiner 함수는 반드시 결합법칙을 만족하고, 간섭받지 않고, 상태를 갖지 않아야 한다."


- 하지만 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다

## 좋은 사용
- n보다 작거나 같은 소수의 개수를 계산하는 함수
```java
static long pi(long n) {
   return LongStream.rangeClosed(2, n)
      .mapToObj(BigInterger::valueOf)
      .filter(i -> i.isProbablePrime(50))
      .count();
}
static long pi(long n) {
   return LongStream.rangeClosed(2, n)
      .parallel()
      .mapToObj(BigInteger::valueOf)
      .filter(i -> i.isProbablePrime(50))
      .count();
}
```


## 결론
1. 계산도 올바로 수행하고 성능도 빨라질 것이라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라. 스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다. 
2. 앞으로 여러분이 스트림 파이프라인을 병렬화할 일이 적어질 것처럼 느껴졌다면, 그건 진짜 그렇기 때문이다. 병렬화가 효과를 보는 경우가 많지 않음을 알게 된다. 
3. 병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능지표를 유심히 관찰하라.
그래서 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때, 오직 그럴 때만 병렬화 버전 코드를 운영 코드에 반영하라. 

# [Item 46] 스트림에서는 부작용 없는 함수를 사용하라
- 스트림 패러다임의 핵심은 계산을 일련의 변환(transform) 으로 재구성하는 것
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수함수여야 한다.
-  순수함수 : 입력만이 결과에 영향을 주는 함수. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이를 위해선, 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다. 

## 좋지않은 사용 
- 텍스트 파일에서 단어별 수를 세어 빈도표로 만드는 코드

```
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
   words.forEach(word -> {
      freq.merge(word.toLowerCase(), 1L, Long::sum);
   });
}
```
- 스트림, 람다, 메서드 참조를 사용했고 결과도 올바르지만 스트림 코드라 할 수 없다. 스트림 코드를 가장한 반복적 코드이다. 
- 길고, 어렵고, 유지보수에 좋지 않다. 
- forEach에서 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다. 

## 좋은 사용
```
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
   freq = words
         .collect(groupingBy(String::toLowerCase, counting()));
}
```
- 같은 일을 하지만 짧고 명확하다.
- forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다. 대놓고 반복적이라서 병렬화할 수도 없다. forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자. 
- 병렬처리 : 한가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리하는 것을 말한다.
- 별개로, groupingBy()는 스레드에 안전하지 않은 map을 생성하고, groupingByConcurrent()는 스레드에 안전한 ConcurrentMap 을 생성한다. (둘다 Collectors의 정적메소드로 같고 Map을 생성하는 방법도 같은데, 스레드에 안전한지 여부가 다름)


- 빈도표에서 가장 흔한 단어 10개를 뽑아내는 스트림 파이프라인 작성
 
```
List<String> topTen = freq.keySet().stream()
      .sorted(comparing(freq::get).reversed())
      .limit(10)
      .collect(toList());
```

- 중간처리: 매핑, 필터링, 정렬
- 최종처리: 반복, 카운팅, 평균, 총합 등 집계처리수행
  
- 그 외에 36가지 메소드가 있다. 책에서는 toMap을 설명하는데, 이 모든 메소드를 지금 이해하려고 하면 어려운 것 같다. 지금은 이런게 있구나 정도로 이해하고, 나중에 필요할 때 찾아서 쓰는 식으로 하는 것이 좋겠다. 

## 결론
    1.스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 
    2.스트림뿐 아니라 스트림 관련 객체에 건네지는 함수 객체가 부작용이 없어야 한다. 
    3.종단 연산 중 for Each는 스트림이 수행한 계산 결과를 보고할 때만 이용해야한다. 
    4.스트림을 잘 사용하려면 collectors (최종처리_수집)을 잘 알아둬야 한다. 가장 중요한 수집 팩터리는 toList, toSet, toMap, groupingBy, joining 이다. 



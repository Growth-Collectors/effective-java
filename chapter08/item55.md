## 상황: 특정 연산의 리턴값이 "있을수도 없을수도 있다"
- Exception을 발생시키기에 충분히 예외적이지 않음
- null을 리턴하기에는 나중에 이 값이 어딘가로 흘러들어가 NullPointerException 발생시킬까 걱정

## Optional<T> 
- 보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야할 때 T대신 Optional<T>를 쓰면 된다.

## Optional을 사용하지 않는 예제 
```java
public static <E extends Comparable<E>> E max (Collection<E> c) { // 여기에 빈 컬렉션이 들어간다면
	if(c.isEmpty())
    	throw new IllegalArgumentException("빈 컬렉션"); // 예외를 던진다
    
    E result = null;
    for (E e : c )
    	if (result==null || e.compareTo(result) > 0)
        	result = Objects.requireNonNull(e);
            
    return result; // E 타입의 값을 리턴
}
```
## Optional을 사용하는 예제 
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) { // 여기에 빈 컬렉션이 들어간다면
   if (c.isEmpty())
   		return Optional.empty(); // 빈 옵셔널을 리턴한다
        
   E result = null;
   for (E e : c)
   		if(result==null||e.compareTo(result) > 0)
        	result = Objects.requireNonNull(e);
            
   return Optional.of(result); // E타입의 값이 들어있는 옵셔널을 리턴한다 
}
```
## 주의할 점
- 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자. 옵셔널을 도입한 취지를 완전히 무시하는 행위다

## 옵셔널 어디에 쓰이고 있을까?
- 참고로 스트림의 종단 연산 중 상당수가 옵셔널을 반환하도록 짜여져 있다. 
- 아래 코드에서의 max: 스트림에서 가장 큰 값을 가진 요소를 Optional타입으로 반환함
- max 메서드는 값을 찾지 못한 경우 Optional을 반환함
```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
	return c.stream().max(Comparator.naturalOrder());
}
```

## 옵셔널 사용해 보기
- 값이 없을때 값을 지정하는 orElse 사용가능
```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```
- 예외를 던지는 orElseThrow 사용가능
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```
- 값이 채워져 있을거라 확신한다면? get 사용가능 (하지만 만약 Optional에 값이 없다면 NoSuchElementException 이 발생함.)
```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```
- get 쓰고싶다. 하지만 Exception 발생시키긴 싫다면? orElseGet 사용가능
```java
Optional<String> optional = Optional.empty();
String result = optional.orElseGet(() -> "World");
System.out.println(result); // 출력 결과: "World"
```
- 옵셔널이 채워져 있는지 먼저 확인하고 싶다면? 옵셔널의 안전 벨브, isPresent
```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID:" + (parentProcess.isPresent() ?
	String.valueOf(parentProcess.get().pid()) : "N/A"));
```
- map
```java
System.out.println("부모 PID:" +
	ph.parent().map(h-> String.valueOf(h.pid))).orElse("N/A");
```
(NoSuchElementException이 발생해야하는거 아닌가? -> 아님 )
<img width="1166" alt="image" src="https://user-images.githubusercontent.com/8848124/225024654-447a272e-6e17-41da-952c-b98664fcb5e2.png">

- 옵셔널들을 필터링하고 싶다면? filter
```java
streamOfOptionals
	.filter(Optional::isPresnet) // 값이 있는 것들만 스트림에 매핑한다
	.map(Optional::get)
```
- 옵셔널을 스트림으로 변환하고 싶다면? flatMap
```
streamOfOptionals
	.flatMap(Optional::stream) // 옵셔널에 값이 있다면 그 값을 원소로 담은 스트림으로, 값이 없다면 빈 스트림으로
```

## 옵셔널을 사용하지 말아야 하는 경우 
- 컬렉션, 스트림, 배열, 옵셔널 같은 `컨테이너 타입`(다른 객체를 담을 수 있는 객체)은 옵셔널로 감싸면 안 된다.
- 빈 Optional<List>를 반환하기보다는 빈 List를 반환하는게 좋다. 빈 컨테이너를 그대로 반환하면 클라이언트에 옵셔널 처리 코드를 넣지 않아도 된다.
- 옵셔널을 map의 값으로 사용하면 절대 안 된다. 그리 한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 된다. (맵에 키가 없는경우, 키가 있지만 값이 empty optional인 경우)
- 박싱된 기본 타입을 담는 옵셔널은 피하자 (기본 타입 자체보다 무거울 수밖에 없다. 값을 두 겹이나 감싸기 때문이다.)
그래서 자바 API 설계자들은 int, long, double 전용 옵셔널 클래스들을 준비해놨다. 바로 OptionalInt, OptionalLong, OptionalDouble이다.

## 옵셔널을 사용해야 하는 경우
- 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional를 반환한다.

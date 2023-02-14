# Item 30. 이왕이면 제네릭 메서드로 만들라
## 제네릭으로 변경하면 좋은 경우
타입을 매개변수로 받거나, 컬렉션을 받아야 하는 경우 유리하다

## 로 타입 사용 - 수용 불가
```
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```
꺼낼 때 캐스팅이 잘못되면 문제가 생긴

## 제네릭 메서드로 바꾸려면...
접근지시자와 리턴타입 사이에 타입 매개변수를 적어준
```
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

## 항등함수(identity function)를 담은 클래스

### 제네릭 싱글턴 팩터리 패턴
타입에 따라 메서드를 여러 개 만들 필요가 없다.
```
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```
- IDENTITY FM은 그 자체로 함수
- 하나의 (싱글턴) 인스턴스인데 타입이 다르다고 해서 여러 개 정의할 필요가 없고 호출하는 쪽에서 타입을 정하기만 하면 됨


### 재귀적 타입 한정 

```
public static <E extends Comparable<E>> E max(COllection<E> c);
```

타입 한정인 <E extends Comparable>는 "모든 타입 E는 자신과 비교할 수 있다"
보통 Comparable에서만 쓴다
  
  
## 결론
클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기 쉽다.

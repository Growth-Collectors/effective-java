#Item 30. 이왕이면 제네릭 메서드로 만들라
##제네릭으로 변경하면 좋은 경우
타입을 매개변수로 받거나, 컬렉션을 받아야 하는 경우 유리하다

## 로 타입 사용 - 수용 불가
```
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

## 제네릭 메서드로 바꾸려면...
```
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

## 항등함수(identity function)를 담은 클래스

### 제네릭 싱글턴 팩터리 패턴
```
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

### 재귀적 타입 한정 
```
public static <E extends Comparable<E>> E max(COllection<E> c);
```

타입 한정인 <E extends Comparable>는 "모든 타입 E는 자신과 비교할 수 있다"
  
## 결론
클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기 쉽다.

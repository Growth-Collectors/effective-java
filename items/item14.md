# item14. Comparable을 구현할지 고려하라.

추가 예정...



### Comparable 인터페이스
- Comparable을 구현하는 것은 그 클래스의 인스턴스들에 자연적인 순서가 있음을 뜻한다.
- **Comparable을 구현한 객체들의 배열**은 손쉽게 정렬을 수행할 수 있다.
```java
Arrays.sort(a);
```
- 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 수행할 수 있다.
    - 이 외에도 수많은 제네릭 알고리즘과 컬렉션 활용이 가능하다.
- 자바 플랫폼 라이브러리 모든 값 클래스와 열거 타입이 Comparable을 구현한다.

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

### compareTo 메서드
- Comparable의 하나뿐인 메서드
- 두 가지만 제외하면 Object의 equals 와 같다.
    - 단순 동치성 비교에 더해 순서까지 비교할 수 있다.
    - 모든 객체에 전역 동치관계를 부여하는 equals 메서드와 달리, compareTo는 타입이 다른 객체를 신경 쓰지 않아도 된다.
        - 타입이 다른 경우
            - 예외를 던져서 처리
            - 공통 인터페이스를 매개로 구현 객체들을 비교할 수도 있다.

### compareTo 메서드의 일반 규약
- equals의 규약과 비슷하다.
- 주어진 객체의 순서를 비교한다.        
    - 주어진 객체보다 작으면 음의 정수(-1), 크면 양의 정수(1), 같으면 0을 반환한다.
    - 비교할 수 없는 타입의 객체가 주어지면, ClassCastException을 던진다.

1. 반사성
    ```java
    // 모든 x, y에 대해서
    sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
    ```

2. 추이성을 보장해야 한다.
    ```java
    x.compareTo(y) > 0 && y.compareTo(z) > 0 // 이면
    x.compareTo(z) > 0 //이다.
    ```

3. 대칭성
    ```java
    //모든 z에 대해 
    x.compareTo(y) == 0 //이면, 
    sgn(x.compareTo(z)) == sgn(y.compareTo(z)) //이다.
    ```

4. 필수는 아닌 권고 (강력 권장)
    ```java
    (x.compareTo(y) == 0) == (x.equals(y))
    ```
    - Comparable을 구현하지만, 이 권고를 지키지 않은 클래스의 경우에는 그 사실을 명시해야 한다.

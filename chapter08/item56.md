> **자바독 태그의 변천사**
> 
자바 버전에 따라 자바독 태그도 발전해왔다.
자바 5 → @literal, @code
자바 6 → @implSpec
자바 9 → @index

API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

> **@param, @return, @throw**
> 

메서드의 계약(contract)을 완벽히 기술하려면 모든 매개변수에 @param 태그를,
반환 타입이 void가 아니라면 @return 태그를,
발생할 가능성이 있는 (검사든 비검사든) 모든 예외에 @throws 태그를 달아야 한다.
관례상 @param, @return, @thro씨S 태그의 설명에는 마침표를 붙이지 않는다

```java
/**
 * Returns the element at the specified position in this list.
 *
 * <p>This method is <i>not</i> guaranteed to run in constant
 * time. In some implementations it may run in time proportional
 * to the element position.
 *
 * @param  index index of element to return; must be
 *         non-negative and less than the size of this list
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         ({@code index < 0 || index >= this.size()})
 */
E get(int index) {
    return null;
}
```

<img width="806" alt="image" src="https://user-images.githubusercontent.com/8848124/226564271-a72a615f-3c56-4af3-abbf-669800de0280.png">

문서화 주석에 HTML 태그`(<p>와 <i>)`를 사용한 이유

→ 자바독 유틸리티 는 문서화 주석을 HTML로 변환하므로 문서화 주석 안의 HTML 요소들이 최종 HTML 문서에 반영되기 때문

> **{@code}**
> 

`{@code}` 태그는 2가지 효과가 있다.

- 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
- 태그로 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시한다.

> **{@literal}**
> 

`{@literal}`태그는 HTML 마크업이나 자바독 태그를 무시하게 해준다

`{@code}` 태그와 비슷하지만 코드 폰트로 렌더링하지 않는다는 차이

> **@implSpec**
> 

메소드에 대한 정보를 문서에 남겨, 다른 프로그래머에게 그 메서드를 올버라게 재정의 하는 방법을 알려준다.

API 설명에 <,>, &등의 HTML 메타문자를 포함시키려면 {@literal} 태그로 감싼다. 

```java
    /**
     * @throws NullPointerException 이 발생할 수 있습니다. 
     * @implSpec 이 메소드는 null 에 취약 하오니, 인스턴스에 null 발생 시,
     * 메소드 재정의 요망
     */
    default void compareZero(){
        if(getNumber() > 0){
            System.out.println("more than Zero ");
        }else{
            System.out.println("less than Zero ");
        }
    }
```

> **{@summary}**
> 

```java
// 첫 번째 마침표가 나오는 곳까지로 인식
/**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */

// 자바 10부터 추가된 요약 설명 전용 태그인 {@summary} 태그 사용
/**
 * {@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
 */
```

각 **문서화 주석의 첫 번째 문장** 은 해당 요소의 설명 (summary description)으로 간주된다. 요약 설명은 반드시 대상 기능을 **고유하게** 기술해야 함

한 클래스(혹은 인터페이스) 안에 서 요약설명이 똑같은 멤버(혹은생성자)가둘 이상이면 안 된다.

> **{@index}**
> 

자바 9부터는 자바독이 생성한 HTML 문서에 검색(색인) 기능이 추가되었다.

클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어지며, 원한다면 `{@index}` 태그를 사용해 API에서 중요한 용어를 추가로 색인화할 수 있다.

```
// 자바독 문서에 색인 추가하기 - 자바 9부터 지원
/**
 * This method complies with the {@index IEEE 754} standard.
 */
```

javadoc HTML 페이지의 검색 상자에 "IEEE 754”를 검색해서 찾을 수 있음

> **제네릭 타입, 제네릭 메서드**
> 

제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.

```java
/**
 * 키와 값을 매핑하는 객체, 맵은 키를 중복해서 가질 수 없다
 * 즉, 키 하나가 가리킬 수 있는 값은 최대 1개다.
 * 
 * @param <K> 이 맵이 관리하는 키의 타입
 * @param <V> 매핑된 값의 타입
 */
public interface Map<K, V> { ... }
```

> **열거 타입**
> 

열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.

```java
// 열거 상수 문서화
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

    /** Stringed instruments, such as violin and cello. */
    STRING;
```

> 애너테이션 타입 문서화 (???)
> 

애너테이션 타입을 문서화할 때는 애너테이션 타입 자체와 멤버들에도 모두 주석을 달아야 한다.

```java
// 한글 버전
/**
 * 이 애너테이션이 달린 메서드는 명시한 예외를 던져야만 성공하는
 * 테스트 메서드임을 나타낸다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD) 
public @interface ExceptionTest {
     /**
     * 이 애너테이션을 단 테스트 메서드가 성공하려면 던져야 하는 예외.
     * (이 클래스의 하위 타입 예외는 모두 허용된다.)
     */
    Class<? extends Throwable> value();
}
```

@Retention(RetentionPolicy.RUNTIME)  // <- 애너테이션의 유지 정책을 지정

@Target(ElementType.METHOD) // <- 다른 애너테이션의 적용 대상을 지정

> **스레드 안전성 & 직렬화 가능성**
> 

API 문서화에서 자주 누락되는 설명 2가지가 `스레드 안전성` 과 `직렬화 가능성` 이다.

클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

또한 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.

> @inheritDoc
> 

자바독은 메서드 주석을 '상속'시킬 수 있다.

자바독은 문서화 주석이 없는 API 요소를 발견하면, 가장 가까운 문서화 주석을 찾아준다. 이때 상위 '클래스'보다 그 클래스가 구현한 '인터페이스'를 먼저 찾는다.

또한 `{@inheritDoc}`태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.

클래스는 자신이 구현한 인터페이스의 문서화 주석을 재사용할 수 있다는 뜻이다.

> **자바독 문서 검사**
> 

자바독은 프로그래머가 자바독 문서를 올바르게 작성했는 확인하는 기능을 제공한다.

자바 7에서는 명령줄에서 `-Xdoclint`플래그를 켜주어야 활성화되고, 자바 8부터는 기본으로 작동한다.

또한 `체크스타일(checkstyle)` 같이 자바독을 검사해주는 IDE 플러그인도 있다

좋은 자바독 예시: 구글 구아바의 EvendBus 패키지 

[https://guava.dev/releases/19.0/api/docs/com/google/common/eventbus/package-summary.html](https://guava.dev/releases/19.0/api/docs/com/google/common/eventbus/package-summary.html)
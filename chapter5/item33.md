# Item33 타입 안전 이종 컨테이너를 고려하라.

- 제네릭은 Set\<E>, Map\<K,V> 등의 컬렉션과 ThreadLocal\<T>, AtomicReference\<T> 등의 단일 원소 컨테이너에도 흔히 쓰인다.
    - 이 때 **매개변수화되는 대상**은 원소가 아닌 **컨테이너 자신**이다.
    - 이러한 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수는 제한적이다.
        - 예를 들어 Map에서는 타입 2개, Set은 타입 1개

## 타입 안전 이종 컨테이너
- 더 유연한 수단의 필요성
- 컨테이너의 **키를 매개변수화**한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화 한 키를 함께 제공한다.
    - 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장한다.
- 이런 설계 방식을 **타입 안정 이종 컨테이너 패턴**(type safe heterogeneous container pattern)이라고 한다.

### 예제
```java
// 구현
public class Favorite {
	private Map<Class<?>, Object> favorites = new HashMap<>();

	public <T> void putFavorite(Class<T> type, T instance) {
		favorites.put(Objects.requireNonNull(type), type.cast(instance));
	}
	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type));
	}
}
```
- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용한다.
    - 이 방식이 동작하는 이유는 **class의 클래스가 제네릭**이기 때문이다. 
    - **class 리터럴 타입은 Class 가 아닌 Class\<T>** 이다.
        - 예를 들어, String.class 의 타입은 Class<String> 이다.

> 컴파일 타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고 받는 class 리터럴을 **타입 토큰**이라고 한다. 

```java
// 클라이언트 코드
public static void main(String[] args) {
	Favorites f = new Favorites();
	
	f.putFavorite(String.class, "Java");
	f.putFavorite(Integer.class, 0xcafebabe);
	f.putFavorite(Class.class, Favorites.class);

	String favoriteString = f.getFavorite(String.class);
	int favoriteInteger = f.getFavorite(Integer.class);
	Class<?> favoriteClass = f.getFavorite(Class.class);

	System.out.printf("%s %x %s\\n", favoriteString, favoriteInteger, favoriteClass.getName());
}
```
- 타입 안전 이종 컨테이너(Favorites) 인스턴스는 타입 안전하다. 
    - String을 요청했는데 Integer를 반환하는 일은 절대없다. 
    - 또한 모든 키의 타입이 제각각이라 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다.

### Map<Class<?>, Object>
- Favorites가 사용하는 private 맵 변수의 타입이다.
- 비한정적 와일드카드 타입이지만, 와일드카드 타입이 중첩(nested) 되어 있다.
    - 맵이 아니라 **키가 와일드카드 타입**
    - 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻 
        - 예를 들어, 첫번째는 Class\<String>, 두번째는 Class\<Integer> 등을 가진 수 있다. 
    - 다양한 타입 지원 가능
- 맵의 값 타입은 Object
    - 키와 값 사이의 타입 관계를 보증하지 않는다.
        - 즉, 모든 값이 키로 명시한 타입임을 보증하지 않는다.
    - 키와 값 사이의 타입 링크 정보는 버려진다.
    - 하지만, get 메서드에서 이 관계를 되살린다. (동적 형변환 활용)

#### 동적 형변환
- 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다.
    - cast 메서드 이용
> **cast 메서드**는 형변환 연산자의 동적 버전
- 주어진 인수가 Class 객체가 알려주는 타입의 인스턴스인지를 검사하고, 맞으면 형변환 된 인수를 아니면 ClassCastException을 던진다.

- cast 메소드의 시그니처
    - Class 클래스가 제네릭이라는 이점을 완벽히 활용
    - cast 의 반환 타입은 **Class 객체의 타입 매개변수**와 같다. 
        - 타입 T 로 비검사 형변환을 하는 손실 없이도 Favorites를 타입 안전하게 만드는 비결이다.
```java
public class Class<T> {
	T cast(Object obj);
}
```

### 제약 사항
- Favorites 클래스엔 두 가지 제약사항이 있다. 

#### 첫번째
- 악의적인 클라이언트가 Class 객체를 로 타입((Class)Integer.class)으로 넘기면 타입 안전성이 쉽게 깨진다는 점이다. 
    - 다만, 클라이언트 코드에서는 비검사 경고가 뜰 것이다.

```java
f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoriteInteger = f.getFavorite(Integer.class); // <- ClassCastException 발생
```

- 이는 HashSet, HashMap 등의 일반 컬렉션도 가지고 있는 문제이다. 
    - 아래와 같이 HashSet의 로 타입을 사용하면 HashSet에 String을 넣는건 아주 쉽다.
```java
HashSet<Integer> set = new HashSet<>();
((HashSet)set).add("string");
```

- Favorites가 **타입 불변식을 어기는 일이 없도록 보장하기 위한 방법 (우회로)**
    - put 메서드에서 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 **동적 형변환**을 이용해 확인한다.
```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), Objects.requireNonNull(type.cast(instance)));
}
```
- java.util.Collections에 존재하는 **checkedSet, checkedList, checkedMap 같은 메서드들**은 이 방식을 적용한 **컬렉션 래퍼들**이다.
    - 제네릭을 이용해 Class 객체와 컬렉션의 컴파일타입 타입이 같음을 보장한다.
    - 또한, 내부 컬렉션들을 실체화 한다. (?이해못함)


#### 두번째
- 실체화 불가 타입에는 사용할 수 없다.
    - 즉, String 이나 String[]은 사용할 수 있어도 List<String>이나 List<Integer>는 저장할 수 없다는 이야기다.
    - List<String>용 Class 객체를 얻을 수 없기 때문이다. (List<String>.class -> 문법 오류 발생)
    - List<String> 이나 List<Integer> 나 List.class라는 같은 Class 객체를 공유한다. List<String>.class 와 List<Integer>.class 를 허용하면, 같은 타입 객체 참조를 반환하게 되어 문제의 소지가 된다.
- 해당 제약에는 만족스러운 우회로가 존재하지 않는다.

#### 참고) 슈퍼 타입 토큰  
- 자바 업계의 거장인 닐 개프터가 고안한 방식
- 스프링 프레임워크에서는 ParameterizedTypeReference 클래스로 구현하여 제공하고 있다.
- 실체화 불가인 제네릭 타입을 활용하는데 매우 유용하다. 
- **한계가 존재**하므로 이를 주의해서 사용해야 한다.

```java
List<String> pets = Arrays.asList("개", "고양이", "앵무");
f.putFavorite(new TypeRef<List<String>>(){}, pets);
```

### 한정적 타입 토큰
- 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
- Favorites 가 사용하는 타입 토큰은 비한정적이다. 
- get, put 메서드들이 허용하는 타입을 제한하고 싶은 경우, 한정적 타입 토큰을 사용하면 된다.

#### Annotation API
- 한정적 타입 토큰을 적극적으로 사용하는 예
```java
public <T extends Annotation>
    T getAnnotations(Class<T> annotationType);
```    
- annotationType 인수는 **어노테이션 타입을 뜻하는 한정적 타입 토큰**이다.
    - 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려있다면 그 어노테이션을 반환하고 없다면 null 을 반환한다.
    - 즉, 애너테이션된(annotated) 요소는 **키를 애너테이션 타입으로 갖는 타입 안전 이종 컨테이너**로 볼 수 있다.

- Class<?> 타입의 객체를 getAnnotaion 처럼 한정적 타입 토큰을 받는 메서드에 넘기는 방법?
    - 객체를 Class<? extends Annotation> 으로 형변환 할수도 있지만, 이 형변환은 비검사이므로 컴파일하면 경고가 뜰 것이다.

    - Class 클래스가 형변환을 안전하게 (그리고 동적으로) 수행해주는 인스턴스 메소드를 제공한다. 
        - **asSubclass 메서드**
        - 호출된 인스턴스 자신의 Class객체를 인수가 명시한 클래스로 형변환한다.
            - 형변환 된다는 것은 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 뜻
        - 형변환에 성공하면 인수로 받은 클래스 객체를 반환하고 실패하면 ClassCastException을 던진다.
```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) { 
	Class<?> annotationType = null; // 비한정적 타입 토큰
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch (Exception ex) {
		throw new IllegalArgumentException(ex);
	} 
	return element.getAnnotation(
		annotationType.asSubclass(Annotation.class));
}
```
- 위 코드는 컴파일 시점에는 타입을 알 수 없는 어노테이션을 asSubclass 메소드를 사용해 런타임에 읽어내는 예이다.
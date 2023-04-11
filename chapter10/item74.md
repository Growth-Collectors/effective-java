# 메서드가 던지는 모든 예외를 문서화하라 

메서드가 던지는 예외는 중요한 정보다
따라서 예외를 문서화 하는 것이 중요하다.

## 검사 예외는 따로따로 선언하자
- 검사 예외는 각각 선언한다
- 각 예외가 발생하는 상황을 자바독의 @throws 태그를 활용해 문서화 한다.
- 공통 상위 클래스 하나로 뭉뚱그려 선언하는 일은 삼간다.

**잘못된 방법**
```java
// 잘못 선언한 예
public void testMethod() throws Exception {

}

// or

public void testMethod() throws Throwable {

}
```
- main 메서드는 JVM만 호출하므로 Exception을 던지도록 선언해도 됨

**올바른 방법**
```java
/**
 * @throws IllegalStateException
 */
public void testMethod(String parameter) throws IllegalStateException {
  
}
```

## 비검사 예외도 문서화 하자
비검사 예외는 일반적으로 프로그래밍 오류를 뜻하므로,
무언인지 알려주면 프로그래머는 해당 오류가 나지 않도록 코딩할 수 있다.

### public 메서드라면 비검사 예외를 문서화하자
- public 메서드라면 필요한 전제조건을 문서화해야하며, 그 수단으로 가장 좋은 것이 바로 비검사 예외를 문서화하는 것이다.

- **인터페이스 메서드**에서 비검사 예외를 문서화하는 것이 중요하다. 문서화한 전제조건이 인터페이스의 일반 규약에 속하게 되어 그 인터페이스를 구현한 모든 구현체가 일관되게 동작하도록 해주기 때문이다.

### 비검사 예외는 메서드 선언의 throws 목록에 넣지 않는다
- 메서드가 던질 수 있는 예외를 @throws로 문서화하되, 비검사 예외는 빼야한다. 
- 검사냐 비검사냐에 따라 API 사용자가 해야 할 일이 달라지므로 이 둘을 확실히 구분해주는게 좋다.
자바독 특징 
- 자바독은 메서드 선언의 throws절에 등장하고 메서드 주석의 @throws 태그에도 명시한 예외와 @throws 태그에만 명시한 예외를 시각적으로 구분해준다. 
- 따라서 프로그래머는 어떤 것이 비검사 예외인지 바로 알 수 있다.

## 한 클래스의 많은 메서드가 같은 이유로 같은 예외를 던진다면
- 각각 메서드에 선언하기 보다 그 예외를 클래스 설명에 추가하는 방법을 고려하자 
```java
/**
 * @throws NullPointerException - 모든 메서드는 param에 null이 넘어오면 NullPointerExcetpion을 던진다.
 */
class Dummy throws NullPointerException {

   public void methodA(String param) {
       ...
   }

   public void methodB(String param) {
       ...
   }

   public void methodC(String param) {
       ...
   }
}
```

## 요약
- 메서드가 던질 가능성이 있는 모든 예외를 문서화
- 검사 예외, 추상 메서드, 구현 메서드 모두 문서화 해야함
- 예외를 문서화할 때는 @throws 태그 사용
- 검사 예외만 throws문에 일일이 선언하고, 비검사 예외는 메서드 선언에 기입하지 않는다.

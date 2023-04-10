### 문제점
- 메서드가 저수준 예외를 처리하지 않고 바깥으로 **전파(throw)** 할 경우 
  
  -> 수행하려는 일과 **관련 없는 예외**로 인해 혼란을 불러온다.
- 내부 구현 방식을 드러내어, API를 오염시킬 수 있음
- 구현 방식을 바꾸면 다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수도 있음

## 해결

### 상위 계층에서 예외 번역을 한다
예외 번역(exception translation)
- 상위 계층에서 **저수준 예외**를 잡아 자신의 **추상화 수준에 맞는 예외**로 바꾸는 것


```java
try {
    ... // 저수준 추상화를 이용
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역
    throw new HigherLevelException(...);
}
```
### 저수준 예외가 필요하면 예외 연쇄 사용힌다
예외 연쇄(exception chaining)
- 문제의 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 것
- 별도의 접근자 메서드(Throwable의 getCause 메서드)를 통해 언제든 저수준 예외를 꺼내 볼 수 있다.
```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
    throw new HighetLevelException(cause);
}

// 예외 연쇄용 생성자 
class HigherLevelException extends Exception {
    HigherLevelException(Throwable cause) {
        super(cause);
    }
}

```
고수준 예외의 생성자는 예외 연쇄용으로 설계된 상위 클래스의 생성자에 원인(cause)을 건네주어 Throwable 생성자로 건네 지게 한다.

예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며, 
원인과 고수준 예외의 스택 추적 정보(stack trace)를 통합하게 해준다.

### 예외 번역을 남용하지 말자 
무작정 예외를 전파하는 것보다 예외 번역이 우수한 방법이지만, 남용하면 안된다.
저수준 메서드가 반드시 성공해서 아래 계층에서 예외가 발생하지 않도록 하는 것이 최선이다.

## 핵심 정리
- 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 노출하기 곤란하다면 예외번역 사용
- 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기 좋다.

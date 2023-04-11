# 아이템 72. 표준 예외를 사용하라

- 재사용!
- 상황에 부합한다면 항상 표준 예외를 재사용하자. 
    - API 문서를 참고해 그 예외가 어떤 상황에서 던져지는지 꼭 확인해야 한다.
    - 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 재사용한다. 
    - 더 많은 정보를 제공하길 원한다면 표준 예외를 확장해도 좋다. 
- 단, 예외는 직렬화할 수 있고(아이템 12), 직렬화에는 많은 부담이 따르기 때문에 이 사실만으로 굳이 사용자 정의 예외를 새로 만들지 않아야 할 근거로 적절하다.

### 장점
- 다른 사람이 사용하기 편리한 API를 제공할 수 있다.
    - 컨벤션
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 줄어든다.


### 널리 재사용되는 예외들
예외 | 주요 쓰임
:--: | :--:
IllegalArgumentException | 허용하지 않는 값이 인수로 건네졌을 때 (null은 따로 NullPointerException으로 처리)
IllegalStateException | 객체가 메서드를 수행하기에 적절하지 않은 상태일 때
NullPointerException | null을 허용하지 않는 메서드에 null을 건넸을 때
IndexOutOfBoundsException | 인덱스가 범위를 넘어섰을 때
ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때 
UnsupportedOperationException | 호출한 메서드를 지원하지 않을 때

#### 추가 예시
- IllegalArgumentException
    - 가장 많이 재사용되는 예외로(아이템 49), 호출자가 인수로 부적절한 값을 넘길 때 던지는 예외다. 
    - 반복 횟수를 지정하는 매개변수에 음수가 할당될 때
- IllegalStateException
    - 이 예외는 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 주로 던진다. 
    - 제대로 초기화되지 않은 객체를 사용하려 할 때 던질 수 있다.
- NullPointerException
    - 메서드가 던지는 모든 예외를 잘못된 인수나 상태라고 포괄적으로 생각할 수도 있어, IllegalArgument라고 볼 수도 있겠으나, 그 중 특수한 일부는 따로 구분해서 써야한다. 
    - null 값을 허용하지 않는 메서드에 null을 건네면 관례상 NullPointerException을 던진다.
- IndexOutOfBoundsException
    - NullPointerException과 유사하게 특수한 예로, 어떤 시퀀스의 허용 범위를 넘는 값을 건넬 때, IndexOutOfBoundsException을 던진다.
- ConcurrentModificationException
    - 단일 스레드에서 사용하려고 설계한 객체(혹은 외부 동기화 방식으로 설계한 객체)를 여러 스레드가 동시에 수정하려 할 때 던진다. 
    - 동시 수정을 확실히 검출할 수 있는 안정된 방법은 없어, 이 예외는 문제가 생길 가능성을 알려주는 정도의 역할로 쓰인다.
- UnsupportedOperationException
    - 이 예외는 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다. 
    - 대부분 객체는 자신이 정의한 메서드를 모두 지원하니 흔히 쓰이는 예외는 아니다. 
    - 보통은 구현하려는 인터페이스의 메서드 일부를 구현할 수 없을 때 쓸 수 있다.
    - 예를 들어 원소를 넣을 수만 있는 List 구현체에 대고 누군가 remove 메서드를 호출하면 이 예외를 던질 것이다.

#### 예외 재사용시 일반적 규칙
- 예외의 주요 쓰임이 상호 배타적이지 않은 탓에 종종 재사용할 예외를 선택하기 어려운 경우가 있다.
    - 예를 들어, 인수 값이 무엇이든 어차피 실패한다면 IllegalStateException 사용
    - 그렇지 않으면, IllegalArgumentException

### Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
- 이 클래스들은 추상 클래스라고 생각하자.
    - 다른 예외들의 상위 클래스이므로, 즉 여러 성격의 예외들을 포괄하는 클래스이므로 안정적인 테스트가 불가능하다.

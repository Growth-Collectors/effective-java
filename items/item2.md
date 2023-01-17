## **아이템 2 - 생성자에 매개변수가 많다면 빌더를 고려하라**
- 정적 팩터리와 생성자에 공통적으로 존재하는 제약  
선택적 매개변수가 많을
때 적절히 대응하기 어렵다.
    - ex)
    식품 포장의 영양정보를 표현하는 클래스를 생각해보자. 영양정보는 1 회 내용량， 총 n회 제공량， 1 회 제공량당 칼로리 같은 필수 항목 몇 개와 총 지방， 트랜스지방， 포화지방， 콜레스테롤， 나트륨 뭉
총 20개가 넘는 선택 항목으로 이뤄진다. 그런데 대부분 제품은 이 선택 항목 중 대다수의 값이 그냥 0 이다.


이러한 선택 매개변수가 많을 때 활용할 수 있는 방법들에 대해 알아본다.


### `점층적 생성자 패턴 (telescoping constructor pattern)`
필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 2개까지 받는 생성자 형태로 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식
```java
public class NutritionFacts {
private final int servingSizei // (ml, 1회 제공랑)
private final int servingsi // (회， 총 n회 제공량)
private final int caloriesi // (1회 제공랑당)
private final int fati // (g/ l회 제공량)
private final int sodiumi // (mg /l회 제공량)
private final int carbohydratei // (g/l회 제공량)

public I\lutritionFacts(int servingSize, int servings) {
this(servingSize, servings, Ø)i
}
public NutritionFacts(int servingSize, int servings, int calories) {
this(servingSize, servings, calories, Ø)i
}
public NutritionFacts(int servingSize, int servings , int calories,
int fat) {
this(servingSize, servings, calories, fat , 0);
}
public NutritionFacts(int servingSize, int servings, int calories,
int fat , int sodium) {
this(servingSize, servings, calories, fat , sodium, 0);
}
public NutritionFacts(int servingSize, int servings, int calories,
int fat , int sodium, int carbohydrate) {
this.servingSize servingSize;
this.servings servings;
this.calories calories;
this.fat = fat;
this.sodium sodium;
this.carbohydrate = carbohydrate;
}
```
- 특징  
    매개변수들이 유효한지를 생성자에서만 확인하면 일관성을 유지할 수 있
- 단점
    - 사용자가 설정하길 원치 않는 매개변수까지 포함하기 쉬운데 어쩔 수 없이 그런 매개변수에도 값을 지정해줘야 한다.
    - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 기 어렵다.
- 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다.  
클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고 결국 런타임에 영뚱한 동작을 하게 된다(아이템 51).

<br>
<br>

### `자바 빈즈 패턴(JavaBeans pattem)`
매개변수가 없는 생성자로 객체를 만든 후, 세터 (setter) 메서드들을 호출해서 원히는 매개변수의 값을 설정하는 방식
```java
public class NutritionFacts {
}
// 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
private int servingSize = -1; // 필수; 기본값 없음
private int servings = -1; // 필수; 기본값 없음
private int calories 0;
private int fat = 0;
private int sodium = 0;
private int carbohydrate = 0;
public NutritionFacts() { }
// 서|터 메서드들
public void setServingSize(int val) {servingSize = val; }
public void setServings(int val) { servings = val; }
public void setCalories(int val) { calories = val; }
public void setFat(int val) { fat = val; }
public void setSodium(int val) { sodium = val; }
public void setCarbohydrate(int val) { carbohydrate = val; }
```
- 특징
    코드가 길어지긴 했지만 인스턴스를 만들기 쉽고 그 결과 더 읽기 쉬운 코드가 되었다.
- 단점
    객체 하나를 만들려면 메서드를 여러 개 호출해야 하고 객체가 완
전히 생성되기 전까지는 일관성 (consistency)이 무너진 상태에 놓이게 된다.
=> 버그를 심은 코드와 그 버그 때문에 런타임에 문제를 겪는 코드가 물리적으로 멀리 떨어져 있을 것 이므로 디버깅도 어려워짐.
=> 일관성 이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변(아이템 17)으로 만들 수 없으며 스레드 안전성을 얻으려면 프로그래머가 추가 작업 을 해줘야만한다.
- 대안
생성이 끝난 객체를 수동으로 ‘얼리고(freezing)' 얼리기 전에는 사용할 수 없도록 하는 방법.
하지만 다루기 어려워서 실전에서는 거의 쓰이지 않는다. 쓴다고 하더라도 객체 사용 전에 프로그래머가 freeze 메서 드를 확실히 호출해줬는지를 컴파일러가 보증할 방법이 없어서 런타임 오류에 취약하다.

<br>
<br>

### `빌더 패턴(Builder pattern)[Gamma95]`
- 순서
    1. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적팩터리)를 호출해 빌더 객체를 얻는다.
    2. 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.
    3. 매개변수가 없는 build 메서드를 호출해 드디어 우리에 게 필요한 (보통은 불변인) 객체를 얻는다.
- ex)
    ```java
    public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    public static class Builder {
    // 필수 매개변수
    private final int servingSize;
    private final int servings;
    // 선택 매개변수 - 기본값으로 초기화한다.
    private int calories ø;
    private int fat ø;
    private int sodium = ø;
    private int carbohydrate = ø;
    public Builder(int servingSize, int servings) {
    this.servingSize = servingSize;
    this.servings servings;
    }
    public Builder calories(int val)
    { calories = val; return this; }

    public Builder fat(int val)
    { fat = val; return this; }
    public Builder sodium(int val)
    { sodium = val; return this; }
    public Builder carbohydrate(int val)
    { carbohydrate = val; return this; }
    public NutritionFacts build() {
    return new NutritionFacts(this);
    }
    private NutritionFacts(Builder builder) {
    servingSize = builder.servingSize;
    servings builder.servi ngs;
    calories builder.calories;
    fat = builder.fat;
    sodium builder.sodium;
    ca rbohydrate = builder.carbohydrate;
    }
    ```
    NutritionFacts 클래스는 불변이며 모든 매개 변수의 기본값들을 한곳에 모아뒀다.
    - 플루언트 API(f]uent API) == 메서드 연쇄 (method chaining)
    메서드 호출이 흐르듯 연결된다는 뜻으로 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출 할 수 있는 방식
        - 이 클래스를 사용하는 클라이언트 코드의 모습
        ```java
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(3S). carbohydrate(27).build();
        ```
        이 클라이언트 코드는 쓰기 쉽 고 무엇보다도 읽기 쉽다.
        cf) 유효성 검사 코드
        잘못된 매개 변수를 최대한 일찍 발견하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고 build 메서드가 호출하는 생성자에서 여러 매개 변수에 걸친 불변식(invariant)을 검사하자. 공격 에 대비 해 이런 불변식을 보장하려면 빌더로부터 매개 변수를 복사한 후 해당 객체 필드들도 검사해 야 한다(아이 댐 50).  검사해서 잘못된 점을 발견하면 어떤 매개 변수가 잘못되었는지를 자세 히 알려주는 메시지 를 담아 IllegalArgumentException을 던지면 된다(아이댐 75).
        cf) 불변과 불변식
        - 불변(immutable/immutability)
        어떠한 변경도 허용하지 않는다는 뜻. 주로 변경을 허용하는 가변 (mutable) 객체와 구분하는 용도로 쓰인다. ex) String 객체 - 한번 만들어지연 절대 값을 바꿀 수 없는 불변 객체다.
        - 불변식(invariant)
        프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건을 말한다. 다시 말해 변경 을 허용할 수는 있으나 주어진 조건 내에서만 허용한다는 뜻이다.
        ex) 리스트의 크기는 반드시 0 이상이어야 하니 만약 한순간이라도 음수 값이 된다면 불변 식이 깨진 것이다 또한, 기간을 표현하는 Period 클래스에서 start 필드의 값은 반드시 end 필드의 값보다 앞서야 하므로， 두 값이 역전되면 역시 불변식이 깨진 것이 다(아이템 50 침조) .
    따라서 가변 객체에도 불변식은 존재할 수 있으며 넓게 보면 불변은 불변식의 극단적인예라 할 수있다.

- 특징
    - 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 방식.
    점증적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고， 자바빈즈보다 훨씬 안전하다.
    - (파이과 스칼라에 있는) 명명된 선택적 매개변수(named optional parameters)를 흉내낸것이다.
    - 생성자나 정적 팩터리가 처리해야 할 매개변수가 많을 떄 유용하다.
    매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.
    - 계충적으로 설계된 클래스와 함께 쓰기에 좋다. 각 계층의 클래
스에 관련 빌더를 멤버로 정의하면 추상 클래스는 추상 빌더를， 구체 클래스(concrete class)는 구체 빌더 를 갖게 한다.
    ```java
        // Pizza. Builder 클래스는 재귀적 타입 한정(아이댐 30)을 이용하는 제네릭 타업
        public abstract class Pizza {
    public enum Topping { HAM， MUSHROM， ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;
    abstract static class Builder<T extends Builder<T>> {
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    public T addTopping(Topping topping) {
    }
    }
    toppings.add(Objects.requireNonNull(topping));
    return self();
    abstract Pizza build();
    /* 
     */
    protected abstract T self();
    Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // 아이템 50 참조
    }
    ```
- 시뮬레이트한 셀프 타입(simulated self-type) 관용구  
추상메서드인 self를 추가해서 self 타입이 없는 자바를 위한 우회 방법.
하위 클래스는 추상 메서드인 self를 재정의(overriding)하여 "this"를 반환하도록 해야 한다. 
=> 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다.

    ```java
    /* 코드 2-5 뉴욕피자 */
    public class NyPizza extends Pizza {
    }
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    public static class Builder extends Pizza.Builder<Builder> {
    private final Size size;
    }
    public Builder(Size size) {
    this.size = Objects.requireNonNull(size); // 크기 매개변수를 필수로 받음
    @Override public NyPizza build() {
    return new NyPizza(this);
    }
    @Override protected Builder self() { return this; }
    private NyPizza(Builder builder) {
    super(builder) ;
    size builder.size;
    }

    /* 코드 2-6 칼초네피자 */
    public class Calzone extends Pizza {
    private final boolean saucelnside;
    public static class Builder extends Pizza.Builder<Builder> {
    private boolean saucelnside = false; // 기본값. 소스를 안에 넣을지 말지 선택하는 매개변수를 필수로 받음.
    public Builder saucelnside() {
    saucelnside = true;
    ```
    각 하위 클래스의 빌 더가 정의한 build 메서 드는 해당하는 구체 하위 클 래스
    를 반환하도록 선언한다.
    - 공변 반환 타이핑(covariant return typing)  
    하위 클 래스의 메서 드가 상위 클래스
    의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능
        - 효과  
        클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.

    클라이언트 측 코드로 열거 타입
    상수를 정적 임포트할 경우
    ```java
    NyPizza pizza = new NyPizza.Builder(SMALL)
    .addTopping(SAUSAGE).addTopping(ONI예 ).build();
    Calzone calzone = new Calzone.Builder()
    . addTopping(H매 ).sauceInside().build();
    ```
생성자로는 누릴 수 없는 사소한 이점으로, 빌 더 를 이용하면 가변인수(varangs) 매개변수를 여러 개 사용할 수 있다.  
각각을 적절한 메서드로 나눠 선언하면 된다.  
아니면 메서드를 여러 번 호출하도록 하고 각 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다.  
코드 2-4의 addTopping 메서 드가 이렇게 구현한 예다.

- 상당히 유연하다.  
    빌더 하나로 여러 객체를 순회하면서 만들 수 있고 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.

- 단점
    - 객체를 만들려면 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비 용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다. 
    - 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.
    하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명심하자. 생성자나 정적 팩터리 방식으로 시작했다가 나중에 매개변수가 많아지면 빌더 패턴으로 전환할 수도 있지만 이전에 만들어둔 생성자와 정적 팩터리가 아주 도드라져 보일 것이 다. 그러니 애초에 빌더로 시작하는 편이 나을 때가 많다.

<br>

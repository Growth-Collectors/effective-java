# 아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라

### ordinal 기반 인덱싱

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다.
식물의 생애주기를 열거 타입으로 표현한 LifeCycle 열거 타입을 예로 보자.

```java
public class Plant {

    enum LifeCycle { ANNUAL, PERNNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

이제 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기별로 묶어보자

- 열거 타입의 ordinal을 배열의 인덱스로 사용하는 코드

```java
public static void usingOrdinalArray(List<Plant> garden) {
    Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        plantsByLifeCycle[i] = new HashSet<>();
    }

    for (Plant plant : garden) {
        plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
    }

    for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
        System.out.printf("%s : %s%n", LifeCycle.values()[i], plantsByLifeCycle[i]);
    }
}
```

이 코드는 **`List<Plant>`** 타입의 식물 리스트를 받아서 각 식물의 수명 주기(LifeCycle)에 따라 그룹화한 결과를 출력하는 메소드이다.

- **`plantsByLifeCycle`**배열 초기화: **`Set`**타입의 배열**`plantsByLifeCycle`**을 **`LifeCycle`**
  enum 값의 개수만큼 생성하고, 각 요소를 **`HashSet`**으로 초기화한다.
- 식물 그룹화: **`garden`**
  리스트의 각 식물을 수명 주기(LifeCycle)에 따라 그룹화한다. **`plantsByLifeCycle`** 배열에서 해당 식물의 수명 주기에 해당하는 **`Set`**에 현재 식물을 추가한다.
- 결과 출력: **`plantsByLifeCycle`** 배열을 순회하면서 각 요소(수명 주기에 해당하는 **`LifeCycle`**enum 값) 와 해당 요소에 속한 식물들을 출력한다.

### 해당 코드의 문제점

- 배열은 제네릭과 호환되지 않는다. 따라서 비검사 형변환을 수행해야한다.
- 사실상 배열은 각 인덱스가 의미하는 바를 알지못하기 때문에 출력 결과에 직접 레이블을 달아야 한다.
- 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.

### EnumMap

이러한 단점들을 java.util 패키지의 **EnumMap** 을 사용하여 해결해보자.
EnumMap은 열거 타입을 키로 사용하는 Map 구현체이다.

```java
public static void usingEnumMap(List<Plant> garden) {
    Map<LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(LifeCycle.class);

    for (LifeCycle lifeCycle : LifeCycle.values()) {
        plantsByLifeCycle.put(lifeCycle,new HashSet<>());
    }

    for (Plant plant : garden) {
        plantsByLifeCycle.get(plant.lifeCycle).add(plant);
    }

//EnumMap은 toString을 재정의하였다.
    System.out.println(plantsByLifeCycle);
}
```

- 이전 ordinal을 사용한 코드와 다르게 안전하지 않은 형변환을 사용하지 않는다.
- 결과를 출력하기 위해 번거롭던 과정도 EnumMap 자체가 toString을 제공하기 때문에 번거롭지 않게되었다.
- ordinal을 이용한 배열 인덱스를 사용하지 않으니 인덱스를 계산하는 과정에서 오류가 날 가능성이 존재하지 않는다.
- EnumMap은 그 내부에서 배열을 사용하기 때문에 **내부 구현 방식을 안으로 숨겨서 Map의 타입 안정성과 배열의 성능을 모두 얻어냈다.**

여기서 EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다. 스트림을 사용하면 코드를 더 줄일 수 있다.

```java
public static void streamV1(List<Plant> garden) {
    Map plantsByLifeCycle = garden.stream().collect(Collectors.groupingBy(plant -> plant.lifeCycle));
    System.out.println(plantsByLifeCycle);
}

public static void streamV2(List<Plant> garden) {
    Map plantsByLifeCycle = garden.stream().collect(Collectors.groupingBy(plant -> plant.lifeCycle,
                    () -> new EnumMap<>(LifeCycle.class),Collectors.toSet()));
    System.out.println(plantsByLifeCycle);
}

```

Collectors의 groupingBy 메소드를 이용하여 맵을 구성하였는데,
streamV1 메소드 와 streamV2 메소드의 차이는 groupingBy 메소드에 원하는 맵 구현체를 명시하였는가의 차이다.

V1 메소서드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻는 공간과 성능 이점이 사라진다.

앞서 보았던 EnumMap 버전과 Stream 버전은 동작이 살짝 다르다.
EnumMap 버전은 열거 타입 상수 별로 하나씩 Key를 전부다 만들지만 Stream 버전에선 존재하는 열거 타입 상수만 Key를 만든다.

![https://blog.kakaocdn.net/dn/DwfJs/btq6A9jRb1d/RBIBCzNJ23ToAk7V7xsaGk/img.png](https://blog.kakaocdn.net/dn/DwfJs/btq6A9jRb1d/RBIBCzNJ23ToAk7V7xsaGk/img.png)

### 좀 더 복잡한 EnumMap 사용

다음은 두 가지 상태(Phase)를 전이(Transition)와 매핑하는 예제이다. 
LIQUID에서 SOLID의 전이는 FREEZE가 되고, LIQUID에서 GAS로의 전이는 BOIL이 될것이다. 
이번 예제에서도 Phase나 Transition의 상수의 선언 순서를 변경하거나 새로운 Phase 상수를 추가하는 경우에도 문제가 발생할 수 있다.

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

위의 코드를 EnumMap을 사용해서 수정한다.

```java
import java.util.EnumMap;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {

        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        private static final Map<Phase, Map<Phase, Transition>> transitionMap = Stream.of(values())
                .collect(Collectors.groupingBy(t -> t.from,// 바깥 Map의 Key
                        () -> new EnumMap<>(Phase.class),// 바깥 Map의 구현체
                        Collectors.toMap(t -> t.to,// 바깥 Map의 Value(Map으로), 안쪽 Map의 Key
                                t -> t,// 안쪽 Map의 Value
                                (x,y) -> y,// 만약 Key값이 같은게 있으면 기존것을 사용할지 새로운 것을 사용할지
                                () -> new EnumMap<>(Phase.class))));// 안쪽 Map의 구현체;public static Transition from(Phase from, Phase to) {
            return transitionMap.get(from).get(to);
        }
    }

}

```

위의 코드는 상당히 복잡하다. Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻이다.

이 코드에서 PLASMA를 추가하면 전이 목록에 IONIZE(GAS, PLASMA)와 DEIONIZE(PLASMA, GAS)만 추가하면 끝이다.

```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);
    }

//나머지 코드는 그대로

}
```

나머지는 기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 작다.

실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다.
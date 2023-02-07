# item24. 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스(nested class)
- 다른 클래스 안에 정의된 클래스
- 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외 쓰임새가 존재하는 경우 톱레벨 클래스로 만들어야 한다.
- 종류
    1. 정적 멤버 클래스
    2. (비정적) 멤버 클래스
    3. 익명 클래스
    4. 지역 클래스
-  1을 제외한 나머지는 내부 클래스 inner class에 해당

## 멤버 클래스
- 중첩 클래스를 메서드 밖에서도 사용해야 하거나, 메서드 안에 정의하기에 너무 길다면 멤버 클래스로 만든다.
- 멤버 클래스의 인스턴스가 바깥 인스턴스를 참조한다면 비정적으로 만들고, 그렇지 않다면 정적으로 만든다.

### 정적 멤버 클래스
- 다른 클래스 안에 선언되고, 바깥 클래스outer class의 private 멤버에도 접근할 수 있다는 점 외에는 일반 클래스와 같다.
- 정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용 받는다.
    - 예를 들어, private으로 선언하면, 바깥 클래스에서만 접근할 수 있다.
- 바깥 클래스와 함께 쓰일 때 유용한 **public 도우미 클래스**로 쓰인다.

#### private 정적 멤버 클래스
- 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다.
    - 예를 들어, Map 구현체가 갖는 키-값 쌍을 표현하는 엔트리 객체들
    - 엔트리의 메서드들(getKey, getValue, setValue)이 직접 맵(바깥 클래스 인스턴스)을 사용하지 않는다. 
        - 그러므로 이런 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비이며, private 정적 멤버 클래스가 가장 알맞다.

### (비정적) 멤버 클래스
- **비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.**
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.  
    - 정규화된 this 란? 
        - `클래스명.this` 형태로 바깥 클래스의 이름을 명시하는 용법
- 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다.
    - 보통 이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어진다.
    - 드물게는 직접 바깥 인스턴스의 클래스.new MemberClass(args) 와 같이 생성자를 호출해 수동으로 만들기도 한다.
    - 이러한 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다. 
- **어댑터**를 정의할 때 자주 활용된다.
    - 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용한다.
    - 예를 들어 Map 인터페이스의 구현체들은 보통 컬렉션 뷰(keySet, entrySet, values 메서드가 반환)를 구현할 때 비정적 멤버 클래스를 사용한다.
        ```java
        // 이하 HashMap 내부 구현 코드 중 일부
        /**
        * Returns a {@link Collection} view of the values contained in this map.
        * The collection is backed by the map, so changes to the map are
        * reflected in the collection, and vice-versa.  If the map is
        * modified while an iteration over the collection is in progress
        * (except through the iterator's own {@code remove} operation),
        * the results of the iteration are undefined.  The collection
        * supports element removal, which removes the corresponding
        * mapping from the map, via the {@code Iterator.remove},
        * {@code Collection.remove}, {@code removeAll},
        * {@code retainAll} and {@code clear} operations.  It does not
        * support the {@code add} or {@code addAll} operations.
        *
        * @return a view of the values contained in this map
        */
        public Collection<V> values() {
            Collection<V> vs = values;
            if (vs == null) {
                vs = new Values();
                values = vs;
            }
            return vs;
        }

        final class Values extends AbstractCollection<V> {
            public final int size()                 { return size; }
            public final void clear()               { HashMap.this.clear(); }
            public final Iterator<V> iterator()     { return new ValueIterator(); }
            public final boolean contains(Object o) { return containsValue(o); }
            public final Spliterator<V> spliterator() {
                return new ValueSpliterator<>(HashMap.this, 0, -1, 0, 0);
            }

            public Object[] toArray() {
                return valuesToArray(new Object[size]);
            }

            public <T> T[] toArray(T[] a) {
                return valuesToArray(prepareArray(a));
            }

            public final void forEach(Consumer<? super V> action) {
                Node<K,V>[] tab;
                if (action == null)
                    throw new NullPointerException();
                if (size > 0 && (tab = table) != null) {
                    int mc = modCount;
                    for (Node<K,V> e : tab) {
                        for (; e != null; e = e.next)
                            action.accept(e.value);
                    }
                    if (modCount != mc)
                        throw new ConcurrentModificationException();
                }
            }
        }

        ```
    - Set 과 List 같은 다른 컬렉션 인터페이스 구현들도 자신의 반복자를 구현할 때, 비정적 멤버 클래스를 주로 사용한다.
        ```java
        // LinkedList 내부 반복자 클래스 구현 일부
        private class ListItr implements ListIterator<E> {
            private Node<E> lastReturned;
            private Node<E> next;
            private int nextIndex;
            private int expectedModCount = modCount;

            ListItr(int index) {
                // assert isPositionIndex(index);
                next = (index == size) ? null : node(index);
                nextIndex = index;
            }
            //... 생략
        }
        ```
- 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면, **정적 멤버 클래스**로 만들어야 한다.
    - 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.

#### 주의점
- **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만든다.**
- 바깥 인스턴스로의 **숨은 외부 참조**를 갖게 된다.
    - 참조를 저장하려면, 시간과 공간이 소비된다.
    - 더 심각한 문제로는 **가비지 컬렉션**이 바깥 클래스의 인스턴스를 수거하지 못하는 **메모리 누수**가 생길 수 있다는 점이다.(아이템 7)
    - 그리고 이러한 참조가 눈에 보이지 않으니 원인을 찾기 어렵게 된다.
- 공개된 클래스의 public 이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요해진다.
    - 멤버 클래스 역시 공개 API가 되니, 향후 릴리스에서 static을 붙이면 하위 호환성이 깨진다.

## 그 외
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면, 익명 클래스를 만들고 그렇지 않으면 지역 클래스로 만든다.

### 익명 클래스
- 바깥 클래스의 멤버가 아니다.
    - 멤버와 달리, 쓰이는 시점에 선언가 동시에 인스턴스가 만들어진다. 즉 코드 어디서든 만들 수 있다.
- 오직 비정적 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서도 상수 변수 이외의 정적 멤버는 가질 수 없다.
    - 즉, 상수 표현을 위해 초기화 된 final 기본 타입과 문자열 필드만 가질 수 있다.
- 응용하는데 제약이 많다.
    - 선언한 지점에서만 인스턴스를 만들 수 있다. 
    - instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
    - 여러 인터페이스를 구현할 수도 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 표현식 중간에 등장하므로, 10줄 이하로 짧게 작성해 가독성을 높이는 것을 권장한다.
- **정적 팩터리 메서드**를 구현할 때 익명 클래스를 활용한다.

#### 람다
- 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 익명 클래스를 주로 사용했으나, 이제는 람다가 익명 클래스를 대체한다.

### 지역 클래스
- 네 가지 중첩 클래스 중 가장 드물게 사용한다.
- 지역 변수를 선언할 수 있는 곳이면 어디서든 선언할 수 있다.
- 유효범위도 지역변수와 같다.

#### 다른 중첩 클래스와의 공통점
- 멤버 클래스
    - 이름이 있고, 반복 사용 가능하다.
- 익명 클래스
    - 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다. 
    - 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.
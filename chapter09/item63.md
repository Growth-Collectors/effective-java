
### 63. 문자열 연결은 느리니 주의하라

---

문자열을 연결할 때 성능 문제가 발생할 수 있다는 내용을 다룬다.

문자열은 불변이라서 두 문자열을 연결할 경우, 양쪽의 내용을 모두 복사해야해서 성능이 저하된다.
.

문자열을 연결할 때는 '+' 연산자나 concat 메서드를 사용한다. 

이러한 연산은 내부적으로 새로운 문자열 객체를 생성한다. 

따라서, 문자열을 연결하는 작업이 빈번하게 발생할 경우, 성능 문제를 유발할 수 있다.

.


- 잘못 사용한 예 (느림)
    
    ```java
    public String statement() {
        String result = "";
        for (int i = 0; i < numItems(); i++) {
            result += lineForItem(i); // 문자열 연결
        }
        return result;
    }
    ```
    
    
.

이를 해결하기 위해서는, 문자열을 연결하는 작업이 빈번하게 발생할 경우 StringBuilder나 StringBuffer 클래스를 사용하는 것이 좋다. 

해당 클래스는 내부 버퍼(buffer)를 사용해서 문자열 연결 작업을 수행하므로, 성능 문제를 해결할 수 있다.

.

- StringBuilder를 사용해 개선
    
    ```java
    public String statement2() {
        StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
        for (int i = 0; i < numItems(); i++) {
            sb.append(lineForItem(i));
        }
        return sb.toString();
    }
    ```
    
.

또한, 문자열 연결 작업을 수행할 때는 자주 사용되는 문자열 상수들을 미리 상수로 선언하여 사용하는 것이 좋다.

이렇게 하면, 상수 풀(constant pool)에 이미 있는 상수를 참조하기 때문에 객체 생성이 줄어들어 성능 향상에 도움이 된다.

.

따라서, 성능에 신경써야한다면 

- 문자열을 연결할 때는 성능 문제를 고려하여 StringBuilder나 StringBuffer 클래스를 사용
    - 문자열 연결 연산자(+)를 피하고, StringBuilder의 append 메서드를 사용
- 자주 사용되는 문자열 상수들을 상수로 선언하여 사용
- 문자 배열을 사용하거나, 문자열을 하나씩 처리하는 방법도 있다.

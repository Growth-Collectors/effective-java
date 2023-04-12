# [Item 11] equals를 재정의하려거든 hashCode도 재정의하라
- equals를 구현할 때 hashCode도 함께 만들어야한다. 롬복은 @EqualsAndHashCode 하나의 어노테이션으로 동작한다. 
  
## hashCode 규약
-  equals에 사용되는 정보가 변경되지 않았다면 hashCode는 매번 같은 값을 리턴해야한다. 
 (변경되거나, 애플리케이션을 다시 실행했다면 달라질 수 있다.)
-  객체에 대한 equals가 같다면, hashCode의 값도 같아야한다.
-  두 객체의 equals가 다르더라도 hashCode의 값을 같을 수 있지만 해시테이블 성능을 고려해 다른 값을 리턴하는 것이 좋다.
  
```
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
```

- 해시충돌 Hash collision: 
    * 서로 다른 key들이 같은 hash를 가질 때 발생 
    * key1 != key2 but hf(key1) == hf(key2) 

- 해시코드 구현에 비용이 많이 든다고 판단되면 캐싱하여 사용해도 된다.
```
    private int hashcode;
 ```
- Thread Safety
  * 멀티스레드 환경에서 Thread Safety이란 , 여러개의 스레드가 해당 코드를 동작할 때 예측한대로 동작하는 것.
  * 해시코드 구현 시 여러개의 스레드가 동시에 작업한다면, 한 객체의 값을 엇갈리며 계산하여 다른 값이 나올 수 있다.
  * 따라서 캐싱할때는 Thread safety를 고려해야한다. 





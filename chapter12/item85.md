## 직렬화
직렬화: 객체 -> 바이트 스트림
역직렬화: 바이트 스트림 -> 객체

## 왜 직렬화를 사용하는가?
- 자바 프로그램 **외**적으로도 객체를 사용하기 위해서 직렬화를 사용한다
  - 직렬화를 통해 객체의 상태를 파일이나 데이터베이스에 저장할 수 있음
  - 네트워크 통신: 객체를 원격으로 전송하거나 원격 메서드 호출(RMI)과 같은 분산 시스템에서 사용할 수 있음

## 직렬화가 왜 문제가 되나?
- 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어렵다는 점이다.
  - 역직렬화시 ObjectlnputStream의 readObject 메서드가 호출되는데,
  - readObject 메서드는 (Serializable 인터페이스를 구현했다면) 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있는 사실상 마법 같은 생성자다.
  - 그렇다면? 악성코드도 충분히 주입될 수 있다는 말 (익스플로잇에 있어서 코드주입이 큰 허들인데, 이걸 손쉽게 해결하게 해줌)
  - "샌프란시스코 교통국을 마비시킨 공격이 정확히 이런 사례로, 가젯들이 체인으로 엮여 피해가 더욱 컸다"

## 역직렬화 폭탄
```java
@DisplayName("역직렬화 폭탄 테스트")
@Test
void deserializeBomb() {
    byte[] bomb = bomb();

    // 역직렬화를 하면 엄청 많은 시간이 걸린다.
    deserialize(bomb);
    assertThat(bomb).isNotEmpty();
}

static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1);
        s1.add(t2);
        s2.add(t1);
        s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // 직렬화 수행
} 
```
1. HashSet 을 역직렬화하려면 원소들의 해쉬코드를 계산해야 한다. 
2. 해쉬코드 계산함수를 2**100번 호출해야 함

![IMG_0343](https://user-images.githubusercontent.com/8848124/235084745-44bfa713-43a9-455f-b97c-68280968db88.jpeg){: width="50%" height="50%"}
=> 200개의 객체만으로 2**100의 call을 유발하는 가성비좋은(?) 공격

## 대처는 어떻게?
- cross-platform structured-data representation 으로 바꿔라
  - 속성-쌍의 집합으로 구성된 간단하고 구조화된 **데이터** 객체 (코드가 아니라서 더 안전한듯) 
  - ex) json, 프로토콜 버퍼(protbuf)
- 근데 레거시 시스템때문에 직렬화를 써야만 한다면?
  - 신뢰할 수 없는 데이터는(외부에서 입력된 데이터) 절대 역직렬화하지 말아라
  - 객체 역직렬화 필터링 (java.io.ObjectlnputFilter) 
    - 스트림이 역직렬화되기 전에 필터를 설치하는 기능
    - 화이트리스트 방식을 추천
    - 그치만 슬프게도 앞에서 보여준 직렬화 폭탄은 걸러내지 못함... (화이트리스트 내의 메서드이기 때문에...)
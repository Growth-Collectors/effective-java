# Item 28. 배열 대신 List를 사용하라
## Generic 요약
### Generic 사용 배경
애초에 왜 Generic이라는 개념이 나왔을까?

Generic 이전에는 Object가 그 자리를 대체했었다.


```
class ObjectBox{
	Object item;
    void setItem(Object item){this.item = item; }
    Object getItem() { return item; }
}
```

크게 문제될 게 없어 보이지만 아니다.

Object로 모든 객체가 들어올 수 있기 때문에 get을 하는 경우 늘 타입체크를 해줘야 했다.

#### Generic 사용 전: Runtime Error
 Generic을 사용하지 않은 List의 경우 디버깅이 어렵다.
 타입 매개변수를 정의하지 않는다면 Object 타입을 받게 된다
 ```

   public class GenericBasic {

    public static void main(String[] args) {

        List numbers = new ArrayList();
        numbers.add(10);
        numbers.add("whiteship");

        for (Object number: numbers) {
            System.out.println((Integer)number);
        }
```

String형을 Integer로 타입캐스팅할 수 없기 때문에 런타임에서 ClassCastException 에러가 난다



#### Generic 사용 이후: Fail fast

    ...
	    List<Integer> nuberms = new ArrayList<>();
        nuberms.add(10);
        nuberms.add("whiteship");

        for (Integer number: nuberms) {
            System.out.println(number);
        }
    }
    ```
    
   타입 변수에 Integer를 썼기 때문에 문자열을 입력하는 순간부터 컴파일 에러가 발생한다.
   

#### 한정적 타입변수 <~ extends>
선언 시 vs 사용 시
선언 시에 타입의 범위를 제한할 수 있다.
아래처럼 Number의 하위타입만 받게 선언했다면, 사용 시점에는 Integer처럼 하위타입만 가능하다.


    public class Box<E extends Number>{
	    private E get(){
		    return this.item;
	    }
		public static void main(String[] args){
			Box<Integer> box = new Box<>();
			box.add(10);
			printBox(box);
		}
    }

만약 main 메서드에서  String 타입 등을 쓴다면 컴파일 에러가 발생한다


	    private static void printBox(Box<String> box){
			System.out.println(box.get());
		}

한편, Object로 선언 시 주의해야 한다.

		private static void printBox(Box<Object> box){
			System.out.println(box.get());
		}

Object 배열과 달리 Object 타입 매개변수를 쓴 리스트의 경우 반드시 Object 타입만 넣을 수 있다.(하위타입 불가능)




#### 비한정적 와일드카드 '?'
Object 배열이 하위타입 호환 가능하듯, 와일드카드를 쓰면 특정 클래스 하위 타입을 넣을 수 있다.
한편, 와일드카드 사용 시 <? extends Object> 같은 케이스는 와일드카드로 대체할 수 있다. 


## 배열과  List의 차이
### Array vs ArrayList
| 특징 |배열|리스트  |
|--|--|--|
| 요소 |primitive+object  | object only |
|  |Generic XXX  |Generic 사용가능 |
|실체화  |O  |X |
| 타입 함께 변화 |O  |X |


 
## 배열과 Generic의 차이

### 공변 vs 불공변
#### 디버깅이 어려운 공변(배열)

```
Object[] objectArray = new Long[1];
/* ArrayStoreException 발생 */
objectArray[0] = "String은 타입이 달라 넣을 수 없음";
```
런타임이 되어서야 실패한다.

#### 디버깅이 쉬운 불공변(List)
```
List<Object> objectList = new ArrayList<Long>();
objectList.add("타입이 달라 넣을 수 없음");
```

### 실체화
배열은 자신이 담고 있는 원소의 타입을 런타임에도 알고 있다
제네릭은 컴파일러에게 표시해주는 용도이므로, 컴파일러가 적힌 타입대로 컴파일 시점에 타입캐스팅을 하게 된다. 

## Generic 배열을 만든다면?

```
public static void main(String[] args) {
  List<String>[] stringLists = new List<String>[1];	// (1) List<String>을 원소로 담고 있는 배열
  List<Integer> intList = List.of(42);				// (2)
  Object[] objects = stringLists;						// (3)
  objects[0] = intList;								// (4)
  String s = stringLists[0].get(0);					// (5)
}
```
(3) : (1)에서 생성한 List<String>을 원소로 담고 있는 배열을 Object 타입의 배열 Objects에 할당한다. 

List<String> 또한 Object이기 때문에 할당할 수 있다. 
(4) : (2)에서 생성한 List<Integer>의 인스턴스를 Objects의 첫 원소로 저장한다.

 **제너릭은 소거방식이기 때문에 컴파일 시 List의 타입이 단순히 List가 되고, 공변 항목에서 다뤘듯 List 자체는 Object 타입이므로 `ArrayStoreException`이 발생하지 않는다.**

이렇게 되면 List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에 intList의 인스턴스가 저장돼 있다.
컴파일러는 List<Integer>의 Integer 타입에 맞추어 get(0) 시점에 타입캐스팅을 하는데,
이를 할당받는 s의 타입은 String이므로 
(5)에서 `ClassCastException`이 발생한다.

## 결론, 요약
- 제네릭은 런타임에 타입 소거되며, 배열은 타입을 기억한다
- 제네릭은 런타임의 ClassCastException을 방지하기 위해서 만들어졌다
- 그러므로 제네릭 배열을 만든다면 타입 안정성(컴파일 시점의 체크)도 해치고, 굳이 만든다면 컴파일러가 타입을 보장해주지 못하므로 unchecked exception 경고를 띄운다. 
- 다만 성능을 위해 컬렉션 라이브러리에서는 제네릭 배열을 사용하기도 한다. (뒷장에 나올 예정)


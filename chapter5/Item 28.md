# Item 28. 배열 대신 List를 사용하라

### Generic 사용 전: Runtime Error
 Generic을 사용하지 않은 List의 경우 디버깅이 어렵다.
 타입 매개변수를 정의하지 않는다면 Object 타입을 받게 된다

   public class GenericBasic {

    public static void main(String[] args) {

        List numbers = new ArrayList();
        numbers.add(10);
        numbers.add("whiteship");

        for (Object number: numbers) {
            System.out.println((Integer)number);
        }
String형을 Integer로 타입캐스팅할 수 없기 때문에 런타임에서 에러가 난다
ClassCastException


### Generic 사용 이후: Fail fast

    ...
	    List<Integer> nuberms = new ArrayList<>();
        nuberms.add(10);
        nuberms.add("whiteship");

        for (Integer number: nuberms) {
            System.out.println(number);
        }
    }
   타입 변수에 Integer를 썼기 때문에 문자열을 입력하는 순간부터 컴파일 에러가 발생한다.
   

###  Generic이란?
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



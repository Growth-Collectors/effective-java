# [Item 12] toString을 항상 재정의하라
## 왜?
* println(), printf(), + 연산자 등 toString() 메서드는 알게 모르게 많이 불려진다.
* System.out.println(phoneNumber)와 같이 작성하면 내부 내용을 볼 수 있으므로 디버깅하기 쉽다.
* 만약 사용자에게 값을 보여줘야 할 일이 있을 때도 간단하게 print(phoneNumber)와 같이 넘기기만 하면 toString()은 자동으로 불려진다. 즉, 개발 시 편리하다.  
## 규약
1) toString은 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
```
object.toString 
-> PhoneNumber@adbbd          VS         Jenney=707-867-5309
```
2) 객체가 가진 모든 정보를 보여주는 것이 좋다.
3) 값 클래스라면 포맷을 문서에 명시하는 것이 좋다
    * 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
    * 단, 포맷을 한번 명시하면, 평생 그 포맷에 얽매이게 된다.
    * 해당 포맷으로 객체를 생성할 수 있는 정적 팩터리나 생성자를 제공하는 것이 좋다. 
4) toString이 반환한 값에 포함된 정보를 얻어올 수 있는 api를 제공하는 것이 좋다.
```
    /**
     * 전화번호의 문자열 표현을 반환한다.
     * xxx-yyy-zzzz 와 같은 형식으로 반환된다.
     * 만일, 각 자리 숫자의 갯수보다 적은 숫자를 넣게 되면 그 공간은 0으로 채워진다.
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
```
5) 재정의가 필요치 않는 경우를 제외하곤 모든 클래스에서 toString을 재정의 하는게 좋다.
   * 정적 유틸리티 클래스는 toString()을 제공할 필요가 없고, enum은 대부분 자바가 이미 완벽한 toString()을 제공한다.


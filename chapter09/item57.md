# **지역변수의 유효 범위를 줄이기**


> ****가장 처음 쓰일때 선언하라****
> 
- 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 ‘가장처음 쓰일 때 선언 하기’다. 사용하려면 멀었는데 미리 선언부터 해두면 코드가 어수선해져 가독성이 떨어진다. 변수를 실제로 사용하는 시점엔 타입과 초깃값이 기억나지 않을 수도 있다.
- 거의 모든 지역변수는 선언과 동시에 초기화 해야한다. 초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야 한다.
    - 예외: try-catch
    - 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록안에서 초기화해야 한다

> 반복 변수의 값을 반복문이 종료된 뒤에도 써야 하는 상황이 아니라면 while문보다 는 for 문을 쓰는 편이 낫다.
> 
- 컬렉션이나 배열을 순회하는 권장 관용구

```sql

// 권장
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) { 
  Element e =i.next();
  ... // e와 i로 무언가를 한다.
}

// 비권장
Iterator<Element> i = c.iterator(); 
while (i.hasNext()) {
  doSomething(i.next()); 
}

Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // 버그! 새로운 반복변수 i2를 초기화했지만, 실수로 이전 while 문에서 쓴 i1를 다시 씀
  doSomethingElse(i2.next());
}
```

for 문을 사용하면 이런 오류를 컴파일 타임에 잡아줄 것이다.

```sql
for (Iterator<Element> i = c.iterator(); i.hasNext()) { 
// 변수 유효 범위가 for 문 범위와 일치하여 
// 똑같은 이름의 변수를 여러 반복문에서 써도 서로 아무런 영향을주지 않는다.
	doSomething(i.next()); 
}

...

// 다음 코드는 "i를 찾을 수 없다"는 컴파일 오류를 낸다. 
while (Iterator<Element> i2 = c2.iterator(); i.hasNext()) {
	doSomething(i2.next());
}
```


> ****메서드를 작게 유지하고 한 가지 기능에 집중하라****
> 

한 메서드에서 여러 가지 기능을 처리한다면 그중 한 기능과 관련된 지역 변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다. 해결책은 간단하다. 단순히 메서드를 기능별로 쪼개면 된다.
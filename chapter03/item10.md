# [Item 10] equals는 일반 규약을 지켜 재정의하라
- 자바에서 모든 클래스는 암묵적으로 object 클래스를 상속한다. 
- 3장은 final이 아닌 메소드 "equals, hashcode, toString, clone, finalize"를 오버라이딩할 때 지켜야할 점에 대해 이야기한다.
- 그 중 equals는, "만들지 않아도 되면 굳이 만들지 않는 것이 좋다."
- 특히 아래 4가지 경우는 더욱 그렇다. 
  1) 각 인스턴스가 본질적으로 고유한 경우 (Singleton, Enum, Thread)
  2) 인스턴스의 논리적 동치성 ('logical equality')를 검사할 일이 없는 경우 (String "Hello")
  3) 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 경우 (list, set: 부모클래스인 abstract list, abstract map에 잘 정의되어 있음)
  4) 클래스가 private or package-private이고 equals 메서드를 호출할 일이 없는 경우 (밖에서 마음대로 참조할 수 없는 클래스인 경우 굳이 equals를 사용하지 않는다. 접근제한자가 public인 경우 누구나 참조해서 쓸 수 있기 때문에 어떻게 쓰일지 예측할 수 없다.)

## equals 재정의 규약
1) 반사성 (A.equals(A) == true)
2) 대칭성 (A.equals(B) == B.equals(A))
3) 추이성 (A.equals(B) && B.equals(C), A.equals(C))
4) 일관성 (A.equals(B) == A.equals(B))
5) null이 아님 (A.equals(null) == false)

```
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    } //반사성 

    if (!(o instanceof Point)){
        return false;
    } //instanceof로 타입비교

    Point p = (point)o; //타입 형변환

    return p.x == x && p.y == y; //핵심필드 같은지 비교 
    //double,float.compare(), nullable인 경우: objects.equals() 
        
}
```

## AutoValue
```
  <dependency>
            <groupId>com.google.auto.value</groupId>
            <artifactId>auto-value-annotations</artifactId>
            <version>1.9</version>
        </dependency>
```

```
@AutoValue
abstract class Point {
    static Point create(int x, int y) {
        return new AutoValue_Point(x, y);
    }

    abstract int x();
    abstract int y();
}
```
lombok? @EqualsAndHashCode

# try-finally보다는 try-with-resources를 사용하라
## (지양) try-finally 
- 자원을 여러개 사용하게 된다면 코드가 지저분해진다 
- ```
  public void play() throws IOException {
    try {
        JavableBook book = new JavableBook();        //----(1)
        try {
            JavableVideo video = new JavableVideo(); //----(1)
            book.page(150);                          //----(2)
            video.scene(150);                        //----(2)
        } finally {
            video.close()                            //----(3)
    } finally {
        book.close()                                 //----(3)
    }
  }
- 어디서 에러가 났는지 추적이 어렵다 (finally 의 구조적 특징)
- ```
  public class AppRunner {
	public static void main(String[] args) {
		MyResource myResource = new MyResource();
		try {
			myResource.doSomething();                // ----(1)
		} finally {
			myResource.close();                      // ----(2)
		}
	}
  }
  ```
  - try 블록과 finally 블록에서 모두 예외가 발생할 수 있는데, 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.
  - 그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어, 실제 시스템에서의 디버깅을 어렵게 한다.
  - finally 의 execption도 [추적하고 싶다면](https://stackoverflow.com/a/57673852)?
## (권장) try-with-resources
```
public void play() throws IOException {
    try (JavableBook book = new JavableBook();
         JavableVideo video = new JavableVideo();){ //----(1)
         book.page(150);
         video.scene(150);                          //----(2)
    }
}
```
- (3)의 단계가 보이지 않음 
  - try-with-resource 에서는 구절이 모두 끝나게 된다면 자동으로 자원을 반납하기 때문
- 숨겨진 예외들도 그냥 버려지지는 않고, 스택 추적 내역에 ‘숨겨졌다 (suppressed)’는 꼬리표를 달고 출력됨 
- 짧고 읽기 수월함

참고 https://tecoble.techcourse.co.kr/post/2021-04-26-try-with-resource/
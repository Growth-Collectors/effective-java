# effective-java-study
## 참고 링크

- [이펙티브 자바 공식 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [이펙티브 자바 3판 번역 용어 설명](https://docs.google.com/document/d/1Nw-_FJKre9x7Uy6DZ0NuAFyYUCjBPCpINxqrP0JFuXk/edit)

## 진행방식
- 인원은 10명으로 구성해 매주 2명씩 정리(마크다운 및 깃헙 활용) 및 발표
    - 각 발표자는 3개의 아이템을 할당받아 정리 및 발표 수행
- 화요일 8시마다 스터디 수행
- 불참시 벌금 3000원 (보증금에서 차감)
- **발표 일자에 불참 불가** 
  - 필요시 다른 분과 발표 일정 변경해주세요

## 발표 및 토론
- 발표자가 정리한 내용을 바탕으로 발표 수행
- **아이템 별로 전체 스터디원이 참여하며 생각 공유 및 내용 정리**
    - 아이템 별로 전체 스터디원이 이슈에 기억에 남는 구절 등을 공유(필수입니다!)
    - 아이템별 이슈에서 자유롭게 소통해주셔도 좋아요!
    
## 2장. 객체 생성과 파괴
| 아이템 | 발표자
:---| :---:
[아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라](chapter02/item1.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라](chapter02/item2.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라](chapter02/item3.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라](chapter02/item4.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](chapter02/item5.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 6. 불필요한 객체 생성을 피하라](chapter02/item6.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 7. 다 쓴 객체 참조를 해제하라](chapter02/item7.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 8. finalizer와 cleaner 사용을 피하라](chapter02/item8.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 9. try-finally보다는 try-with-resources를 사용하라](chapter02/item9.md) | [eternalklaus](https://github.com/eternalklaus)

## 3장. 모든 객체의 공통 메서드
| 아이템 | 발표자
:--- | :---:
[아이템 10. equals는 일반 규약을 지켜 재정의하라](chapter03/item10.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 11. equals를 재정의하려거든 hashCode도 재정의하라](chapter03/item11.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 12. toString을 항상 재정의하라](chapter03/item12.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 13. clone 재정의는 주의해서 진행하라](chapter03/item13.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 14. Comparable을 구현할지 고려하라](chapter03/item14.md) |[HanaHww2](https://github.com/HanaHww2)

## 4장. 클래스와 인터페이스
| 아이템 | 발표자
:--- | :---:
[아이템 15. 클래스와 멤버의 접근 권한을 최소화하라](chapter04/item15.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라](chapter04/item16.md) | [jioome](https://github.com/jioome)
[아이템 17. 변경 가능성을 최소화하라](chapter04/item17.md) | [jioome](https://github.com/jioome)
[아이템 18. 상속보다는 컴포지션을 사용하라](chapter04/item18.md) | [jioome](https://github.com/jioome)
[아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.](chapter04/item19.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 20. 추상 클래스보다는 인터페이스를 우선하라](chapter04/item20.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라](chapter04/item21.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라](chapter04/item22.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라](chapter04/item23.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 24. 멤버 클래스는 되도록 static으로 만들라](chapter04/item24.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라](chapter04/item25.md) | [kimrumm](https://github.com/kimrumm) 

## 5장. 제네릭
| 아이템 | 발표자
:--- | :---:
[아이템 26. 로 타입은 사용하지 말라](chapter05/item26.md) | [kimrumm](https://github.com/kimrumm)
[아이템 27. 비검사 경고를 제거하라](chapter05/item27.md) | [kimrumm](https://github.com/kimrumm)
[아이템 28. 배열보다는 리스트를 사용하라](chapter05/item28.md) | [Tldkt](https://github.com/Tldkt)
[아이템 29. 이왕이면 제네릭 타입으로 만들라](chapter05/item29.md) | [Tldkt](https://github.com/Tldkt)
[아이템 30. 이왕이면 제네릭 메서드로 만들라](chapter05/item30.md) | [Tldkt](https://github.com/Tldkt)
[아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라](chapter05/item31.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라](chapter05/item32.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 33. 타입 안전 이종 컨테이너를 고려하라](chapter05/item33.md) | [HanaHww2](https://github.com/HanaHww2)

## 6장. 열거 타입과 애너테이션
| 아이템 | 발표자
:--- | :---:
[아이템 34. int 상수 대신 열거 타입을 사용하라](chapter06/item34.md) | 
[아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라](chapter06/item35.md) | 
[아이템 36. 비트 필드 대신 EnumSet을 사용하라](chapter06/item36.md) |
[아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라](chapter06/item37.md) | [kimrumm](https://github.com/kimrumm)
[아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라](chapter06/item38.md) | [kimrumm](https://github.com/kimrumm)
[아이템 39. 명명 패턴보다 애너테이션을 사용하라](chapter06/item39.md) | [kimrumm](https://github.com/kimrumm)
[아이템 40. @Override 애너테이션을 일관되게 사용하라](chapter06/item39.md) | [jioome](https://github.com/jioome)
[아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라](chapter06/item39.md) | [jioome](https://github.com/jioome) 

## 7장. 람다와 스트림
| 아이템 | 발표자
:--- | :---:
[아이템 42. 익명 클래스보다는 람다를 사용하라](chapter07/item42.md) | [jioome](https://github.com/jioome)
[아이템 43. 람다보다는 메서드 참조를 사용하라](chapter07/item43.md) | [Tldkt](https://github.com/Tldkt)
[아이템 44. 표준 함수형 인터페이스를 사용하라](chapter07/item44.md) | [Tldkt](https://github.com/Tldkt)
[아이템 45. 스트림은 주의해서 사용하라](chapter07/item45.md) | [Tldkt](https://github.com/Tldkt)
[아이템 46. 스트림에서는 부작용 없는 함수를 사용하라](chapter07/item46.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다](chapter07/item47.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 48. 스트림 병렬화는 주의해서 적용하라](chapter07/item48.md) | [juyeong-repo](https://github.com/juyeong-repo)

## 8장. 메서드
| 아이템 | 발표자
:--- | :---:
[아이템 49. 매개변수가 유효한지 검사하라](chapter08/item49.md) | [yeGenieee](https://github.com/yeGenieee) 
[아이템 50. 적시에 방어적 복사본을 만들라](chapter08/item50.md) | [yeGenieee](https://github.com/yeGenieee) 
[아이템 51. 메서드 시그니처를 신중히 설계하라](chapter08/item51.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 52. 다중정의는 신중히 사용하라](chapter08/item52.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 53. 가변인수는 신중히 사용하라](chapter08/item53.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라](chapter08/item54.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 55. 옵셔널 반환은 신중히 하라](chapter08/item55.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라](chapter08/item56.md) | [eternalklaus](https://github.com/eternalklaus)

## 9장. 일반적인 프로그래밍 원칙
| 아이템 | 발표자
:--- | :---:
[아이템 57. 지역변수의 범위를 최소화하라](chapter09/item57.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 58. 전통적인 for 문보다는 for-each 문을 사용하라](chapter09/item58.md) | [kimrumm](https://github.com/kimrumm)
[아이템 59. 라이브러리를 익히고 사용하라](chapter09/item59.md) | [kimrumm](https://github.com/kimrumm)
[아이템 60. 정확한 답이 필요하다면 float와 double은 피하라](chapter09/item60.md) | [kimrumm](https://github.com/kimrumm)
[아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라](chapter09/item61.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라](chapter09/item62.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 63. 문자열 연결은 느리니 주의하라](chapter09/item63.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 64. 객체는 인터페이스를 사용해 참조하라](chapter09/item64.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 65. 리플렉션보다는 인터페이스를 사용하라](chapter09/item65.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 66. 네이티브 메서드는 신중히 사용하라](chapter09/item66.md) | [juyeong-repo](https://github.com/juyeong-repo)
[아이템 67. 최적화는 신중히 하라](chapter09/item67.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 68. 일반적으로 통용되는 명명 규칙을 따르라](chapter09/item68.md) | [Yerimi11](https://github.com/Yerimi11)

## 10장. 예외
| 아이템 | 발표자
:--- | :---:
[아이템 69. 예외는 진짜 예외 상황에만 사용하라](chapter10/item69.md) | [Yerimi11](https://github.com/Yerimi11)
[아이템 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라](chapter10/item70.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 71. 필요 없는 검사 예외 사용은 피하라](chapter10/item71.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 72. 표준 예외를 사용하라](chapter10/item72.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 73. 추상화 수준에 맞는 예외를 던지라](chapter10/item73.md) | [jioome](https://github.com/jioome)
[아이템 74. 메서드가 던지는 모든 예외를 문서화하라](chapter10/item74.md) | [jioome](https://github.com/jioome)
[아이템 75. 예외의 상세 메시지에 실패 관련 정보를 담으라](chapter10/item75.md) | [jioome](https://github.com/jioome)
[아이템 76. 가능한 한 실패 원자적으로 만들라](chapter10/item76.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 77. 예외를 무시하지 말라](chapter10/item77.md) | [YunDaHyee](https://github.com/YunDaHyee)

## 11장. 동시성
| 아이템 | 발표자
:--- | :---:
[아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라](chapter11/item78.md) | [YunDaHyee](https://github.com/YunDaHyee)
[아이템 79. 과도한 동기화는 피하라](chapter11/item79.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 80. 스레드보다는 실행자, 태스크, 스트림을 애용하라](chapter11/item80.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 81. wait와 notify보다는 동시성 유틸리티를 애용하라](chapter11/item81.md) | [HanaHww2](https://github.com/HanaHww2)
[아이템 82. 스레드 안전성 수준을 문서화하라](chapter11/item82.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 83. 지연 초기화는 신중히 사용하라](chapter11/item83.md) | [yeGenieee](https://github.com/yeGenieee)
[아이템 84. 프로그램의 동작을 스레드 스케줄러에 기대지 말라](chapter11/item84.md) | [yeGenieee](https://github.com/yeGenieee)

## 12장. 직렬화
| 아이템 | 발표자
:--- | :---:
[아이템 85. 자바 직렬화의 대안을 찾으라](chapter12/item85.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 86. Serializable을 구현할지는 신중히 결정하라](chapter12/item86.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 87. 커스텀 직렬화 형태를 고려해보라](chapter12/item87.md) | [eternalklaus](https://github.com/eternalklaus)
[아이템 88. readObject 메서드는 방어적으로 작성하라]() | 
[아이템 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라]() | 
[아이템 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라]() | 

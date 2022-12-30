# effective-java-study
## 참고 링크

- [이펙티브 자바 공식 Github](https://github.com/WegraLee/effective-java-3e-source-code)
- [이펙티브 자바 3판 번역 용어 설명](https://docs.google.com/document/d/1Nw-_FJKre9x7Uy6DZ0NuAFyYUCjBPCpINxqrP0JFuXk/edit)

## 진행방식
- 인원은 10명으로 구성해 챕터별로 2명씩 정리(마크다운 및 깃헙 활용) 및 발표
    - 발표자는 3개의 아이템을 할당받아 정리 및 발표 수행
- 화요일 8시 이후 저녁 시간대에 스터디 수행
- 불참시 벌금 3000원
- 발표 일자에 불참 불가 
  - 필요시 다른 분과 발표 일정 변경해주세요

## 발표 및 토론
- 발표자가 정리한 내용을 바탕으로 발표 수행
- 아이템 별로 전체 스터디원이 참여하며 생각 공유 및 내용 정리

## 2장. 객체 생성과 파괴
| 아이템 | 발표자
:---: | :---:
[아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라]() | 
[아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라]() | 
[아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라]() | 
[아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라]() | 
[아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라]() | 
[아이템 6. 불필요한 객체 생성을 피하라]() | 
[아이템 7. 다 쓴 객체 참조를 해제하라]() | 
[아이템 8. finalizer와 cleaner 사용을 피하라]() | 
[아이템 9. try-finally보다는 try-with-resources를 사용하라]() |

## 3장. 모든 객체의 공통 메서드
| 아이템 | 발표자료
:---: | :---:
[아이템 10. equals는 일반 규약을 지켜 재정의하라]() | 
[아이템 11. equals를 재정의하려거든 hashCode도 재정의하라]() |
[아이템 12. toString을 항상 재정의하라]() | 
[아이템 14. Comparable을 구현할지 고려하라]() |

## 4장. 클래스와 인터페이스
| 아이템 | 발표자료
:---: | :---:
[아이템 15. 클래스와 멤버의 접근 권한을 최소화하라]() | 
[아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라]() | 
[아이템 17. 변경 가능성을 최소화하라]() | 
[아이템 18. 상속보다는 컴포지션을 사용하라]() | 
[아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라.]() | 
[아이템 20. 추상 클래스보다는 인터페이스를 우선하라]() | 
[아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라]() | 
[아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라]() | 
[아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라]() | 
[아이템 24. 멤버 클래스는 되도록 static으로 만들라]() | 
[아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라]() | 

## 5장. 제네릭
| 아이템 | 발표자료
:---: | :---:
[아이템 26. 로 타입은 사용하지 말라]() |
[아이템 27. 비검사 경고를 제거하라]() | 
[아이템 28. 배열보다는 리스트를 사용하라]() | 
[아이템 29. 이왕이면 제네릭 타입으로 만들라]() | 
[아이템 30. 이왕이면 제네릭 메서드로 만들라]() | 
[아이템 31. 한정적 와일드카드를 사용해 API 유연성을 높이라]() | 
[아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라]() | 
[아이템 33. 타입 안전 이종 컨테이너를 고려하라]() | 

## 6장. 열거 타입과 애너테이션
| 아이템 | 발표자료
:---: | :---:
[아이템 34. int 상수 대신 열거 타입을 사용하라]() | 
[아이템 35. ordinal 메서드 대신 인스턴스 필드를 사용하라]() | 
[아이템 36. 비트 필드 대신 EnumSet을 사용하라]() |
[아이템 37. ordinal 인덱싱 대신 EnumMap을 사용하라]() | 
[아이템 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라]() |
[아이템 39. 명명 패턴보다 애너테이션을 사용하라]() | 
[아이템 40. @Override 애너테이션을 일관되게 사용하라]() | 
[아이템 41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라]() | 

## 7장. 람다와 스트림
| 아이템 | 발표자료
:---: | :---:
[아이템 42. 익명 클래스보다는 람다를 사용하라]() | 
[아이템 43. 람다보다는 메서드 참조를 사용하라]() |
[아이템 44. 표준 함수형 인터페이스를 사용하라]() | 
[아이템 45. 스트림은 주의해서 사용하라]() | 
[아이템 46. 스트림에서는 부작용 없는 함수를 사용하라]() | 
[아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다]() |
[아이템 48. 스트림 병렬화는 주의해서 적용하라]() | 

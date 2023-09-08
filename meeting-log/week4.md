# 4week issue에 대한 토론 일지

---
## 회의 시나리오
복붙용

## 이슈

1. 안건을 올린 이유 발표
  
2. 안건에 대해 답변자 발표 및 설명 요약
  
3. Q n A 및 회의록 기록

---
## 이슈

1. 안건을 올린 이유 발표
- Java의 null 처리 방법에는 Optional 이외에도 requiredNonNull, assert 등이 있는 것으로 알고 있습니다.
이들과 Optional은 각각 어떻게 사용되는지 차이점이나 특징, 사용사례 등에 대해 알고싶습니다.
  
2. 안건에 대해 답변자 발표 및 설명 요약
- 자바에서 null은 아주 골칫덩어리 : 자바 애플리케이션에서 가장 많이 발생하는 예외가 NPE 발생.
- null을 피하려면?
  - 매번 분기 ? : 잊을 수도 있고, 너무 코드가 복잡해짐 => null handling 필요함.
- null handling 방법 :
  - 선언과 동시에 초기화.
  - Objects.isNull(), Objects.nonNull() 사용 : if(var == null) 사용 지양 권장.
  - Objects.requireNonNull() : 가독성을 위함.
  - 타입으로 관리하기 : 레퍼런스 타입만 null을 가지기 때문에 되도록 primitive 타입 사용 권장.
  - 메서드 오버로딩 : 오버로딩을 활용하여 필요한 인자만 들어가는 메서드를 정의할 것.
  - Collections.emptyList() : Collection을 반환하는 메서드에서는 Collections.emptyList() 반환.
  - Optional :
    - null을 요소로 넣지 말 것 -> empty 활용.
    - ofNullable()을 사용하여 null 처리할 것.
- OPtional 사용시 주의사항 :
  - 멤버 변수에 Optional 선언은 안티 패턴 : 직렬화 불가 -> 메서드 반환 목적으로만 사용할 것.
  - collection에는 optional을 사용하지 말것 : Collections.empty..() 활용할 것.
  
5. Q n A 및 회의록 기록


---
## 이슈

1. 안건을 올린 이유 발표
  책에서 모듈성에서 리플렉션을 언급하면서 모듈을 읽어오는 것을 얘기했지만 강력한 기능임에도 언급만 하고 넘어가 이번 기회에 리플렉션에 대해서 알아보면 좋을 것이라고 생각함
2. 안건에 대해 답변자 발표 및 설명 요약
리플렉션?

- 객체의 클래스를 몰라도 클래스의 메서드, 멤버변수에 접근할 수 있게해주는 자바 API

리플렉션으로 가져올 수 있는 정보

- 동적으로 런타임에 클래스를 추출할 수 있어서 컴파일 상황에서 에러가 발생하지 않음
- 인텔리제이에서 자동완성, 스프링 어노테이션에서 리플렉션이 사용됨
- 클래스, 생성자, 메서드를 가져오는 기능을 제공하여 클래스의 정보를 모두 가져올 수 있음

리플렉션을 사용하면 접근제어자 상관없이 모든 필드에 접근할 수 있다
- 모듈에서는 우리의 클래스에 대한 리플렉션을 사용할 수 있는지 여부를 명시적으로 표현할 수 있음

자바에서의 사용
- 디버깅, JUnit과 같은 테스트 프레임워크에서 런타임에 알 수 없는 객체의 동작 과정 분석시
- third party 라이브러리를 사용하는 경우 해당 라이브러리에 존재하는 클래스 및 메서드를 분석 하는 경우

  
3. Q n A 및 회의록 기록


---
## 이슈

1. 안건을 올린 이유 발표
   
```java
public String getCarInsuranceName(Optional<Person> person) {
 return person.flatMap(Person::getCar)
 .flatMap(Car::getInsurance)
 .map(Insurance::getName) // (1)
 .orElse
```

(1)에서 Optional<Insurance>가 사용되어서 Optional<Optional<String>>가 반환될 줄 알았는데 Optional<String>이 반환된 이유를 알고싶습니다. Optional<Optional<>>로 반환되는지 Optional<>로 반환되는지 판단 기준을 알고 싶습니다.

2. 안건에 대해 답변자 발표 및 설명 요약

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) { }
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) { }
```

- map
  - 반환값은 U를 상속하는 클래스에 Optional을 감싸서 반환
  - mapper의 function
- flatMap
  - mapper의 반환 타입 그대로 반환
  
4. Q n A 및 회의록 기록


---


---

## 이슈 추상 클래스와 인터페이스, Optional orElse 차이

1. 안건을 올린 이유 발표
  - 자바 8을 기준으로 차이를 생각해보면 좋겠습니다.
  - optional의 종단 메서드를 쓰는 기준을 생각해보면 좋겠습니다.

2. 안건에 대해 답변자 발표 및 설명 요약
  - 자바 8 이후에는 구현부가 있는 정적 메서드랑 디폴트 메서드가 있기 때문에 추상 클래스 처럼 구현부를 가진 메서드를 자식에게 줄 수 있게됐다.
  - 차이점은 추상 클래스는 멤버 인스턴스 변수를 사용할 수 있고, 단일 상속만 지원한다.
  - 인터페이스는 다중 상속을 지원합니다.
  - optional 내부 값이 null이 아닐 때 orElse는 함수를 받을 때 실행하고 orElseGet은 인자로 받은 함수를 실행하지 않습니다. 
  
3. Q n A 및 회의록 기록

---
## 이슈

1. 안건을 올린 이유 발표
  
2. 안건에 대해 답변자 발표 및 설명 요약
  
3. Q n A 및 회의록 기록

---

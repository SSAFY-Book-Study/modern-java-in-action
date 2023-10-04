# 4장 매일 자바와 함께

## 1. Null 대신 Optional 클래스

### 1.1 배경

- 대부분의 언어에서 null 참조 개념을 포함.
    - 예전 언어와의 호환성 유지를 위함.
    - 구현이 쉬워지기 때문.
- null 참조로 인한 실질적인 피해가 큼.
    - **에러(NullPointerException)의 근원**이 될 수 있음.
    - null 확인 코드/ 방지 코드로 인한 코드 **가독성 저하**.
    - 정적형식 언어에서 값이 없음 을 표현한 값으로 null(아무의미 없음)은 **적절하지 않음**.
        - 정적형식 언어 : 자료형이 고정된 언어. 컴파일을 진행할 때, 변수의 타입이 결정됨.
    - 자바는 포인터를 개발자로 부터 숨겨야 하는데(자바의 철학), **null pointer는 숨겨지지 X**.
    - 시스템 모든 곳에 null이 퍼지기 쉬워짐으로써, **애초에 null이 어떤 의미로 사용되었는지 파악 어려움**.

---

### 1.2 다른 언어에서의 null 처리 방법

- Groovy : 안전 네비게이션 연산자.
    - 호출 체인에 null이 있으면 참조값으로 null 반환.
- Haskell : Maybe라는 선택형 값(값 or nothing) 저장.
- Scala : Option[T]라는 아무런 값도 갖지 않는 형태 저장.

---

### 1.3 기존 Java의 null 처리 방법

1. null 확인 코드 추가(깊은 의심)
    - 문제점 : 반복패턴 코드로 인한 들여쓰기 수준이 증가 ⇒ 가독성 저하.
2. null인 경우를 처리(너무 많은 출구)
    - 문제점 : 반복패턴 코드로 인한 너무 많은 출구 생성 ⇒ 협업하는 누군가 null임을 잊을 수 있음.

|  | 기존 Java | 다른 언어 |
| --- | --- | --- |
| Null 처리 방법 | 근본적 해결 X, 문제상황을 뒤로 미룸 | 시스템 형식을 통해 null 처리 강제 |

---

### 1.4 Java.util.Optional<T>

- **API 메서드 시그니처만 보고도** 선택형 값인지 아닌지 **판단**가능.
- 선택형 값을 가루는 새로운 클래스(**선택형 값을 캡슐화**).
    - 값 O : 값을 Optional 클래스로 감싸줌.
    - 값 X : Optional.empty() 메서드로 Optional 반환.
- null vs Optional :
    - null : 참조시 NullPointerException 발생.
    - Optional : 객체인 덕분에 예외발생 X.
- 모델이 의미(**semantic**)이 더욱 **명확**해짐.
    - 값이 없는 상황에서 우리 데이터의 문제인지 알고리즘의 버그인지 확인가능.
    - 언랩해서 값이 없는 상황에 적절한 대응가능.

---

### 1.5 Optinal 적용 패턴

1. Optional 객체 생성
    1. **Optinal.empty()** : 빈 Optional 객체 생성.
    2. **Optional.of()** : null이 아닌 값 표현. null이 들어오면 컴파일에러(vs 기존 : 런타임에러).
    3. **Optional.ofNullable()** : null 값 저장 가능. 감싸진 객체가 null이라면 빈 Optional 객체 반환.
2. Optional 객체 정보 추출 : Optional의 명시적인 검사 제거(클래스의 기능을 통해 null 여부 검사).
    1. **Optinal.map(객체::getter)** : Optional에 포함된 객체가 1개라면 인자로 받은 함수가 값 변경, 0개라면 아무일도 발생 X.
    2. **Optinal.flatmap(객체::getter)** : 중첩 Optinal 구조에서 flatmap 사용시, 2차원 Optinal → 1차원.

---

### 1.6 Optional 사용 장점

- null 확인용 조건 분기 필요 X ⇒ 코드가 **간결**해지고, **명확**해짐.
- 도메인 모델과 관련된 암묵적인 지식에 의존하지 않아도 됨 ⇒ 명시적으로 **형식 시스템이 정의**해주기 때문.

---

### 1.7 도메인 모델의 필드로서 Optional

- 원래 Optional은 선택적 반환값을 지원하는 용도 : 도메인 모델에의 사용은 원래 의도가 아님.
- 도메인 모델에 Optional이 사용되지 않을 것이라고 생각하고 만들었기 때문에 : Optinal 클래스는 **Serializable 을 구현하지 X**.
- 그럼에도 Optional이 가진 장점이 강력하기 때문에 도메인 모델에도 Optional 사용 추천.
    - 도메인 필드에 Optional을 정의할 경우 직렬화 문제 해결 : 내부 필드를 반환받는 메서드**(getter)가 Optional을 반환하도록** 함.

---

### 1.8 Optional stream 조작

- Optional.stream() :
    - Optional을 포함하는 스트림 메서드를 쉽게 처리.
    - Optinal 스트림 → 값을 가진 스트림 변환시 유용.
- flatmap(Optional::Stream) : filter() 빈값 제거 와 map() Optional 제거를 동시에 처리.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
	return persons.stream()
                .map(Person::getCar) //List<Person> -> Optional<car>
                .map(optCar -> optCar.flatMap(Car::getInsurance) //Optinal<Car> -> Optional<Insurance>
                .map(optIns -> optIns.map(Insurance::getName) //Optional<Insurance> -> Optional<String>
                .flatMap(Optinal::stream) //Optinal<String> -> Stream<Optinal<String>> -> Stream<String>
                .collect(toSet());
}
```

---

### 1.9 디폴트 액션과 Optional 언랩

- Optional 인스턴스에 포함된 값을 읽는 다양한 방법 :
    1. **get()** :
        - 래핑된 값 → 해당 값.
        - 빈값 → NoSuchElementException.
    2. **orElse()** :
        - 빈값 → 기본값(인자) 제공.
    3. **orElseGet()** :
        - 빈값 → Supplier 메서드(인자) 실행.
        - orElse vs orElseGet :
            - orElse : 기본값을 미리 생성.
            - orElseGet : 해당하는 값이 없을 때만 Supplier 메서드 생성(게으른 버전).
    4. **orElseThrow()** :
        - 빈값 : 예외 발생(인자를 통해 발생시킬 예외지정).
    5. **ifPresent()** :
        - 래핑된 값 : 인수로 넘겨준 Consumer 실행.
    6. **ifPresentOrElse()** :
        - 래핑된 값 : 인수로 넘겨준 Consumer 실행.
        - 빈값 : 인수로 넘겨준 메서드 실행.
- filter 로 특정값 거르기 :
    - **filter(Predicate)** 실행시
        - 빈값 : 동작 X.
        - 래핑된 값 : Predicate 결과 false인 경우, 값은 사라지고 Optional 상태가 됨.

---

### 2.0 Optional 실용 예제

- 잠재적으로 null이 될 수 있는 값 안전하게 감싸기.
- 값이 없을 때 예외 발생시키기.
- stream 기본형 특화 클래스 vs Optional 기본형 특화 클래스 :
    - Optional은 최대 요소의 개수가 1개이므로 성능 향상 X.
    - Optional 기본형 특화 클래스로 생성된 결과는 Optional로 생성된 결과와 혼용 X.
    - Optional 기본형 특화 클래스에는 map, flatMap, filter X.

## 2. 새로운 날짜와 시간 API

### 2.1 배경

- Java 8 이전의 날짜와 시간 API 문제점 : 직관적 X. thread-safe X.

---

### 2.2 활용

1. **LocalDate** : 시간을 제외한 날짜를 포현하는 불변객체.
    - TemporalField : 시간 관련 객체에서 어떤 필드의 값에 접근할 지 정의할 인터페이스.
        - ChronoField가 구현체.
        - Temporal 인터페이스 구현 : 특정시간을 모델링하는 객체의 값을 어떻게 읽고 조작할 지 정의.
    - 문자열 파싱 : DateTimeFormatter을 이용해서 문자열 → LocalDate.
2. **LocalTime** : 시간을 표현.
    - 문자열 파싱 : DateTimeFormatter을 이용해서 문자열 → LocalDate.
3. **LocalDateTime** : LocaDate와 LocalTime을 쌍으로 갖는 복합 클래스.
4. **Instant** : 유니크 에포크 시간(1970년 1월 1일 0시 0분 UTC)기준으로 특정지점까지를 초로 표현(기계적인 관점).
    - Instant.ofEpochSecond() : 팩토리 메서드.
    - 나노초(1/10억) 정밀도 제공.
    - 사람이 읽을 수 있는 시간정보 제공 X.
5. **Duration** : 두 시간객체(LocalDateTime or Instant) 사이의 지속시간.
    - Duration.between() : 정적 팩토리 메서드로 생성.
6. **Period** : 두 시간객체 사이(LocalDate)의 지속시간.
    - Period.between() : 정적 팩토리 메서드로 생성.

---

### 2.3 날짜 조정, 파싱, 포맷팅

1. withAttribute
2. 상대적인 방식으로 날짜 조정 : Temporal & TemporalAdjuster
3. DateTimeFormatter : 날짜나 시간을 특정 형식의 문자열로 표현.
    - LocalDateTime → LocalDateFormat() → String.
    - String → parse → LocalDateTime
    - thread-safe.
    - 정적 팩토리 메서드 제공.

## 3. 디폴트 메서드

### 3.1 배경

- 인터페이스 규칙 : 인터페이스를 상속받는 클래스는 인터페이스가 가진 메서드를 모두 구현(override)하거나 슈퍼클래스의 구현(override)을 상속받아야 한다.
- 문제상황 : 인터페이스를 수정(새로운 기능 추가)할 경우, 이를 상속받은 모든 클래스도 함께 수정해주어야 한다.
- 문제상황 상세과정 :
    1. 인터페이스에 새로운 기능 추가.
        1. 기존 클래스 파일을 컴파일 하면 정상동작.
        2. 기존 클래스 파일에서 새로운 기능을 호출하면 ⇒ 런타임 에러 발생.
        3. 이후부터 기존 클래스 파일을 재빌드하면 ⇒ 컴파일 에러 발생.

    <aside>
    💡 문제상황에서의 호환성 문제

    - 바이너리 호환성 유지 O : 기존 클래스 파일에서 새로운 기능을 호출하지 않는 한, 정상작동.
    - 소스 호환성 유지 X : 기존 클래스 파일에서 새로운 기능을 호출하면 에러 발생.
    - 동작 호환성 유지 X : 기존 클래스 파일에서 새로운 기능을 호출하면 똑같은 입력을 넣어도 에러 발생.

  ⇒ 원인 : 바이너리 호환성까지는 새롭게 기능을 추가한 인터페이스까지만 재빌드를 하면 되니까 호환성이 유지된다. 그러나 기존 클래스 파일을 수정하는 순간, 해당 소스코드도 재빌드를 해야하기 때문에 소스 호환성과 동작 호환성이 유지되지 않는 것.

    </aside>


---

### 3.2 디폴트 메서드

- 해결방안 : Java 8부터는 **메서드 구현을 포함하는 인터페이스**를 정의할 수 있다.
    - 메서드 구현을 포함한다 : 기존의 인터페이스에는 구현이 없는 추상 메서드만 작성 가능했으나, Java 8부터는 구현부가 있는 메서드 작성이 가능해짐.
- 인터페이스의 구현부가 있는 메서드 :
    - 정적 메서드 : 기존 정적 메서드를 인터페이스에도 사용할 수 있도록 함.
        - 정적 메서드 & 유틸리티 클래스 :
            - Java 8 이전까지는 인터페이스에 정적 메서드를 추가할 수 없었기 때문에 특정 인터페이스나 그 인스턴스를 활용하는 정적메서드들을 모아둔 유틸리티 클래스를 생성하고, 편하게 활용하였음.
              Ex. Collection 인터페이스의 유틸리티 클래스 Collections.
            - Java 8 이후로는 인터페이스 자체에 정적 메서드를 추가할 수 있으므로, 굳이 유틸리티 클래스를 분리하여 생성하지 않고 **인터페이스 자체를 유틸리티 클래스로 활용 가능**해짐.
              Ex. Comparator.NaturalOrder() 등.
              그럼에도 이전 버전과의 호환성을 위해 여전히 몇몇 유틸리티 클래스는 존재.
    - **디폴트 메서드** : 자신을 구현하는 클래스에서 메서드를 반드시 구현하지 않아도 되도록 그 구현부를 인터페이스에서 작성할 수 있도록 함.

    ```java
    public interface Sizable {
    	int getSize();
    	/* 디폴트 메서드 시그니처 : default 키워드 사용 */
    	default boolean isEmpty() {
    		return getSize() == 0;
    	}
    }
    ```

    - 인터페이스 vs 추상 클래스 :
        - Java 8 이후로, 인터페이스에도 인스턴스 메서드와 정적 메서드를 사용할 수 있게되었다면, 기존의 추상 클래스와 동일하지 않나?
        - **다중상속**의 차이 : 추상클래스는 다중상속이 안되고 인터페이스는 가능.
        - **변수 선언** 차이 : 인터페이스는 불가 (static final 변수만 가능 - 아무 인스턴스도 존재하지 않는 시점이기 때문에).

---

### 3.3 디폴트 메서드 활용 패턴

- **선택형 메서드** :
    - 똑같은 인터페이스를 구현하는 클래스들 중 인터페이스의 추상 메서드가 필요하지 않은 경우에는 해당 메서드의 내용을 구현하지 않고 빈 구현으로 남겨둠 ⇒ 불필요한 코드 존재.
    - 많은 수의 자식 클래스들이 필요하지 않은 기능은 그냥 디폴트 메서드로 정의한다면, 해당 기능이 필요없는 구현체들은 그냥 해당 메서드를 오버라이드 하지 않으면 됨 ⇒ **불필요한 코드를 줄일 수 있다**.
- **동작 다중 상속** :
    - 디폴트 메소드가 생기면서 동작을 추가한 메소드를 상속받을 수 있기 때문에 단순히 기능(메서드 시그니처)만 상속받는 것이 아니라, 동작(구현부) 다중 상속이 가능해지게 됨.
    - 다만, 여러개의 동작을 동시에 상속받을 수 있으므로, 중복되는 메서드 시그니처를 상속받는다면 어떤 메서드를 상속받을지 충돌을 해결하기 위한 해석 규칙이 필요함 ⇒ **유연성 제공**.
    - 동작 다중 상속 규칙(해석 규칙) :
        1. **클래스 우선** 적용 : 인터페이스 vs 클래스 ⇒ 클래스의 메서드 우선 적용.
        2. **서브 인터페이스 우선** 적용 : 부모 인터페이스 vs 자식 인터페이스 ⇒ 자식 우선 적용.
        3. 1,2번으로도 우선순위를 결정할 수 없는 경우(충돌상황) : 상속받는 클래스에서 해당 메서드를 **명시적으로 오버라이드** 해야함(충돌문제해결 : 명시적으로 어떤 것을 사용할지 결정).
- 델리게이션 :
    - 상속 vs 위임(delegation) :
        - 상속 : is a ~ 관계인 경우.

          Ex. Person ← Student : Student는 Person이며, Person이 가진 대부분의 기능과 속성을 가진다.

        - **위임 : has a ~ 관계**인 경우.

          Ex. Service { Repository.save() } : Service에서는 Repository의 저장, 삭제, 조회 등의 기능만 필요한데 이를 위해 모든 Repository의 기능 및 속성들을 상속받는 것은 오히려 비효율적.

            - 디폴트 메서드를 여러 클래스에서 델리게이션하여 사용할 경우, 디폴트 메서드의 기능이 변경되지 않도록 **final** 로 선언.

        ```java
        public class UserServiceImpl {
        	private UserRepostiory userRepository; //내부 기능에 접근하기 위해 인스턴스 생성.
        
        	/**
                 * 회원 서비스 클래스에서 회원가입 로직을 구현하려면 UserRepository의 save() 기능 필요.
                 * 그러나 이 때문에 모든 UserRepository의 기능을 상속받는 것은 오히려 비효율적.
                 * 이처럼 단순히 어떠한 클래스(인터페이스)의 특정 기능만 필요한 경우, 위임 활용.
                 * */
        	public void save(User user) {
        		userResitory.save(user);
        	}
        }
        ```


## 4. 자바 모듈 시스템

### 4.1 배경

> *추론하기 쉬운 소프트웨어는 개발 생산성을 높인다.*
>
- 추론하기 쉬운 소프트웨어 :
    - 고수준 모듈 : **의미 있는 기능을 제공**하는 모듈 ⇒ 추론하기 쉬운 모듈.
    - 저수준 모듈 : 고수준 모듈의 기능을 구현하기 위해 필요한 **기능의 실제 구현**.
    - Ex. 판매정보 동기화 API

                          [고수준 모듈]

    - 쇼핑시스템에서 판매정보 읽어오기           ⇒
    - 외부 ERP 시스템에 저장하기                   ⇒
    - 처리 결과 기록하기                                 ⇒

                       [저수준 모듈]

    - 쇼핑 DB에서 판매정보 SELECT
    - 외부 ERP API 호출
    - 처리결과 DB에 INSERT
- 개발 생산성을 높인다 :
    - 저수준 모듈(구체적인 구현 방식)을 고수준 모듈(기능)에 의존하도록 하면, 동일한 기능을 구현하는 세부적인 방법이 변경되거나 확장되어도 그 구현체(생성되는 인스턴스)만 변경될 뿐, 코드는 수정되지 X.
    - 즉, **확장, 유지보수, 재사용이 용이**해진다.
- 고수준 모듈을 만드는 방법 :
    1. **관심사 분리** :
        - 프로그램을 고유한 기능(겹치지 않는 코드 그룹)으로 분리.
        - 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업 가능.
        - 개별 부분을 재사용하기 쉬움.
        - 전체 시스템을 쉽게 유지보수 가능.
    2. **정보은닉** :
        - 캡슐화를 통해 세부 구현을 숨기도록 장려.
        - 의도치 않게 코드의 변화가 일어날 영향 감소.

---

### 4.2 Java 9 이전 모듈화의 한계

- Java 모듈화 수준 : class < package < JAR.
1. **제한된 가시성(캡슐화)** :
    - Java 모듈들의 정보은닉 : **코드가 노출**되어 코드를 임의로 조작하는 위험에 노출됨.
        - 클래스 : 접근제한자 & 캡슐화.
        - 패키지 : public 접근제한자 사용가능 → 서로 다른 패키지의 코드에 의존하려면 public 밖에는 방법이 없는데, 그러면 서로가 아닌 다른 모두에게 공개되어버림.
        - JAR : 패키지와 마찬가지. 정보은닉 거의 할 수 없음.
2. **클래스 경로(classpath)** :
    - classpath : 바이트 코드에 포함된 명령어를 실행하려면, 해당 명령어를 가진 파일(.class)을 찾을 수 있어야 하는데, 이때 JVM이나 javac가 참조하는 소스코드 to 바이트코드까지의 경로.
    - JAR & classpath : 컴파일된 모든 클래스파일들을 하나의 JAR에 넣고 classpath에 추가. 필요한 경우, classpath에 정의된 JAR 내부 파일 경로를 확인.
    - 문제점 :
        - 같은 클래스 간의 **버전 구분 X** ⇒ 같은 라이브러리의 다른 버전을 사용하고 싶어도 마음대로 X.
        - **명시적인 의존성 X** ⇒ JAR는 classpath에 추가될 때 하나의 classes 집합으로 추가되기 때문에 특정 파일경로를 알 수 없으므로, 충돌 or 비어있는 경우 상세경로를 알 수 X.
3. **거대한 JDK** :
    - JDK는 발전과정에서 필요없는 기능도 굉장히 많아짐.
        - 컴팩트 프로파일 : 너무 많은 기능을 관련분야에 따라 3가지로 구분하여 메모리 풋프린트를 제공하긴 하지만 완벽하지는 X.

<aside>
💡 모듈화 필요성

1. **강력한 캡슐화** : 클래스 뿐만 아니라 패키지, JAR에도 정보은닉 필요.
2. **쉽게 유추할 수 있는 classpath** : JAR 내부의 클래스들에 대한 명시적인 의존성 필요.
3. **필요한 부분만 골라서 사용할 수 있는 JDK**.

</aside>

---

### 4.3 자바 모듈 활용


![IMG_6200DA713EB5-1](https://github.com/SSAFY-Book-Study/modern-java-in-action/assets/90823532/d0a0c268-3b9a-4c39-b589-1de97b9dea75)


- 모듈 : Java 8에서 제공하는 새로운 구조 단위.
- 모듈 디스크립터 : module-info.java에 저장된 바이너리 형식.
- 활용 방법 :
    - 모듈 정의는 module-info.java 파일에 해야하며, 해당 파일은 src/main/java 디렉터리에 위치해야함.
    - 모듈명에 대한 정확한 규칙 X.
        - 오라클 권장 : 패키지명처럼 인터넷 도메인명을 역순으로 사용.
    1. **exports** 구문
        - 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다.
        - **어떤 패키지를 공개할 것인지 명시**적으로 지정 ⇒ 강력한 캡슐화.
    2. **requires** 구문
        - 컴파일 타임과 런타임에 **한 모듈이 다른 모듈에 의존함을 정의**.
        - java.base 모듈은 기본적으로 모든 모듈이 의존하고 있기 때문에, 이외의 모듈에 대한 의존성 명시.
    3. **requires transitive** 구문
        - 다른 모듈이 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정.
        - 필요로 하는 모듈이 다른 모듈의 형식을 반환하는 상황에서의 **전이성 선언**.
    4. **exports to** 구문
        - **사용자별로 공개할 기능 제한** ⇒ 가시성을 좀 더 정교하게 제어.
    5. **opens & opens to** 구문
        - 모든 패키지를 다른 모듈에 반사적으로 접근 허용.
        - **어떤 패키지에 리플렉션**이 가능하게 할 것인지 선택.
            - 리플렉션 : JVM이 클래스 로더를 통해 클래스 정보를 읽어와서 메모리에 저장함으로써, 클래스에 대한 정보를 상세히 알 수 있음.

    ```java
    module com.example.foo {
    	/* com.example.foo 모듈은 com.example.foo.http을 읽을 수 있음 */
    	requires com.example.foo.http;
      /* com.example.foo 모듈은 com.example.foo.network와 network가 포함하는 모든 모듈을 읽을 수 있음 */
    	requires transitive com.example.foo.network;
    
    	/* com.example.foo.bar 패키지를 공개 */
    	exports com.example.foo.bar;
    	/* com.example.foo.internal 패키지를 com.example.foo.probe에게만 공개 */
    	exports com.example.foo.internal to com.example.foo.probe;
    
    	/* com.example.foo.quux 패키지에 대한 리플렉션 허용 */
    	opens com.example.foo.quux;
    	/* com.example.foo.internal 패키지에 대한 리플렉션을 probe에게만 허용 */
    	opens com.example.foo.internal to com.example.foo.probe;
    
    	/* com.example.foo.spi.Intf를 서비스 소비자로 지정 */
    	uses com.example.foo.spi.Intf;
    	/* com.example.foo.spi.Impl를 서비스 생산자로 지정 */
    	uses com.example.foo.spi.Impl;
    }
    ```

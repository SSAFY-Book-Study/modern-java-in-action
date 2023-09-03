# PART4

- orElse vs orElseGet
- 델리게이션
- 추상 클래스와 인터페이스 차이
- 다이아몬드 문제 c++과 다른 점

---

# 11장 null 대신 Optional

- null 이란 프로그래밍 언어를 만들 때 값이 없는 상황을 가장 단순하게 구현할 수 있기에 만들어졌다. 모든 객체는 null일 수 있기 때문에 프로그래밍에서 피해 비용이 컸다.

## null의 문제

- NullPointerException의 근원적인 문제
- 모든 코드에 null check code가 들어가게 된다.
- 값이 없다를 표현하기엔 애매하다.
- 자바 철학의 위배 : 자바는 개발자에게 모든 포인터를 숨겼지만,  null은 숨기지 못했다.
- 잠재적으로 null이 프로그램에 퍼지게되면 의미를 알 수 없고 리스크가 증가된다.

## Optional

- jdk 8에서 추가된 Null을 표현할 선택형 값을 캡슐화하는 클래스

### null 대신 Optional.empty()를 사용하라.

- null 참조는 NullPointerException이 발생하지만 Optional.empty 를 사용하게되면 다양한 방식으로 활용할 수 있다.
- 명시적으로 값이 없을 수도 있음을 나타낸다.
- Optional은 더 이해하기 쉬운 api설계를 돕는 것이고, 즉 메서드의 시그니처만 보고 선택형값 여부를 구별하기 위함이다.
- 언랩해서 값이 없는 상황에대한 대응을 강제하는 효과를 준다.
- 하지만, 모든 null참조를 optional로 대치하는 것은 바람직하지 않다. 반드시 값을 가져야 하는 변수라면, 예외 처리가 아니고 값이 없는 이유를 밝혀서 문제를 해결해야한다.

## Optional factory

```java
// 빈 객체
Optional.empty();

// null이 아닌 객체
Optional.of(car);

// null일수도 있는 객체, null이라면 빈 객체 반환
Optional.ofNullable(car);
```

## Optional Map()

```java
// stream 처럼 map 연산을 지원한다.
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

- map은 Optional이 값을 갖고 있으면 값을 변경하고 비어있으면 아무 일도 일어나지 않는다.
- ifPresent에 반환값이 있는 Function을 넣는 행위와 비슷함.

- flatMap()
    - Optional으로 map()한 값이 Optional이게 되면  Optional<Optional<Car>> 이런 중첩된 자료형을 갖게되는데 stream에서와 동일하게 동작하는 flatMap으로 해결할 수 있다.

```java
String name = person.flatMap(Person::getCar)
											.Map(Car::getName)
											.orElse("unknown");
```

## Optional의 목적

- 자바에서 공식적으로 Optional은 함수의 반환값으로 사용할 것을 명확하게 못박았다.

> Optional은 필드형식에 사용하지 않아야 하고 Serializable을 구현하지 않았기 때문에 직렬화 될 수 없다.
> 

### 직렬화가 필요한 도메인에 Optional을 적용하기

- 접근 메서드의 반환값으로 구현

```java
public class Person {
	private Car car;
	public Optional<Car> getCarAsOptional() {
		return Optional.ofNullable(Car);
	}
}
```

## Optional Stream

- jdk 9 에서 추가된 요소가 Optional인 stream

```java
// 기존의 변환 방법, 존재한다면 get 하는 방식
list.stream()
		.filter(Optional::isPresent)
		.map(Optional::get)
		.collect(Collectors.toList());

// 추가된 Optional.stream으로 한번에
list.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());

// Optional::stream은 Optional을 값을 갖고 있다면 1개의 요소를 갖는 스트림 값이 없다면 0개의 요소를 갖는 스트림으로 변환한다.
// 이를 FlatMap을 통해 평면 스트림으로 바꾸면 Optional을 건너뛸 수 있다.
```

- Stream과 같이 filter 연산도 사용 가능하다.

## Optional 처리 메서드

```java
// 값을 읽는 간단하지만 안전하지 않은 메서드, 없다면 NoSuchElementException
Optional.get();

// 없다면 other 반환
Optional.orElse(T other);

// lazy버전의 메서드, Optional에 값이 없을 때만 Supplier가 실행된다.
Optional.orElseGet(Supplier);

// Optional이 비어있을 때 지정한 예외를 발생 시킨다.
Optional.orElseThrow(Supplier);

// 존재하면 consumer가 동작
Optional.ifPresent(Consumer);

// 존재하면 Consumer, 없다면 Runnable
Optional.ifPresentOrElse(Consumer, Runnable);
```

### 기본형 Optional은 권장되지 않는다.

> Stream과 달리 요소가 한 개 이므로 성능 개선이 힘들고 일부 유용한 메서드를 지원하지 않는다. 그리고 일반 Optional과 혼용할 수 없기에 호환성도 떨어진다.
> 

# 12장 새로운 날짜와 시간 API

- jdk 8 이전 날짜와 시간 기능 클래스들은 많은 불편함을 초래했다.

### java.util.Date

- 날짜를 의미하는 Date와 달리 특정 시점을 밀로초 단위로 표현한다.
- 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스 등의 모호한 설계로 유용성이 떨어짐
- JVM기본 시간대인 CET(central european time)을 사용함
- DateFormat은 스레드 안정성이 없다.
- 불편하지만 과거 버전과 호환성을 깨지 않으면서 해결할 수 있는 방법이 없었음

### java.util.Calendar

- 자바에서 대안으로 내놓은 클래스
- 달의 인덱스가 0부터 시작하는 등 날짜 관련 클래스가 두 개라 혼란이 왔다.
- 가변 클래스라는 문제점. → 유지보수 관점

---

## java.time package

- jdk 8 에서 추가된 시간, 날짜 관련 기능들이 정의된 패키지
- 자바에서 사용하길 권장하는 클래스

### LocalDate

- 시간을 제외한 날짜를 표현하는 불변 객체

```java
LocalDate date = LocalDate.of(2017, 9, 21); 
int year = date.getYear();
Month month = date.getMonth();
int day = date.getDayOfMonth();
DayOfWeek dow = date.getDayOfWeek();
int len = date.lengthOfMonth();  // 3월의 일 수
boolean leap = date.isLeapYear(); // 윤년

LocalDate.now(); // 시스템 시계 정보를 이용한 현재 날짜 정보 제공
```

- Enum ChronoField로 어떤 값에 접근할지 get() 메서드로 접근 가능

### LocalTime

- 시간을 표현하는 클래스

### LocalDateTime

- LocalDate 와 LocalTime 을 쌍으로 갖는 복합 클래스
- LocalDate나 LocalTime에서 만들 수도 있고 둘을 가지고 만들거나 추출 가능

## Instant

- 기계 관점에서 특정 지점을 하나의 수로 표현하는 시간 표현 방법
- Unix epoch time(1970,1/1, 0:0:0) 기준으로 현재 지점을 초와 나노초로 표현한다.

### Duration

- 두 시간 객체 사이의 지속시간을 나타내는 객체

```java
Duration between = Duration.between(date1, date2);
```

- Temporal인터페이스를 구현하는 LocalTime, LocalDateTime 등으로 만들 수 있다.
- 초와 나노초로 시간을 표현하므로 LocalDate는 쓸 수 없다.

### Period

- 두 시간 객체 사이 지속시간을 년, 월, 일로 시간을 표현할 때 사용한다.
- between 메서드로 두 시간의 차이를 확인할 수 있다.

> 두 기간 객체는 다양한 팩토리 메서드를 제공하고 있어 시간 객체 없이도 다양한 생성을 할 수 있다.
> 

## 날짜 조정, 파싱, 포멧팅

- 시간 객체들을 불변 객체로 소개하기 때문에 스레드 안전성과 일관성이 유지된다.
- 그래서 객체들의 값을 수정한 복사본 객체를 제공하는 다양한 메서드를 제공해준다.

## TemporalAdjusters

- 복잡한 날짜 조정 기능을 제공해주는 정적 팩토리 메서드 집합
- `ex) lastDayOfYear =>`올해의 마지막 날짜를 반환하는 TemporalAdjuster를 반환함
- TemporalAdjuster는 함수형 인터페이스 이므로 람다 등을 통해 커스텀이 가능함.

- 주말은 건너뛰고 날짜를 다음날로 바꾸는 Custom temporalAdjuster lambda

```java
TemporalAdjuster nextWorkingDay = TemporalAdjusters.ofDateAdjuster(
				temporal -> {
            int dayOfWeek = temporal.get(ChronoField.DAY_OF_WEEK);
            int dayToAdd = 1;
						if (dayOfWeek == DayOfWeek.FRIDAY.getValue()) {
                dayToAdd = 3;
            } else if (dayOfWeek == DayOfWeek.SATURDAY.getValue()) {
                dayToAdd = 2;
            }
            return temporal.plus(dayToAdd, ChronoUnit.DAYS);
        });
```

## DateTimeFormatter

- 스레드에서 안전하게 사용할 수 있는 포메터
- DateTimeFormatterBuilder를 이용해서 복잡적인 포매터를 만들 수 있다.

## ZoneId

- 기존 TimeZone을 대체할 수 있는 시간대 클래스
- 서머타임(Daylight Saving Time)같은 복잡한 사항이 자동으로 처리된다.
- ZoneRules 클래스에는 약 40개정도의 시간대가 있다.
    - 지역 id는 지역/도시 형식으로 이뤄지며 iana time zone database에서 제공하는 정보를 사용한다.
- 시간 객체들을 ZonedDateTime으로 변환할 수 있고, 지정한 시간대의 상대적인 시점을 얻을 수 있다.

# 13장 디폴트 메서드

- 자바에서 라이브러리 설계자는 인터페이스에 새로운 메서드를 추가할 때 이전 프로그램들의 호환성을 걱정해야한다. 모든 구현체를 수정해줘야 함.
- jdk 8에서 구현을 포함하는 인터페이스 메소드를 만들어서 해결했다.

### 기본 구현을 포함하는 인터페이스의 메서드

- 인터페이스에 static method
- default method
- 디폴트 메서드를 이용하면 인터페이스의 기본 구현을 그대로 상속하므로 자유롭게 인터페이스에 기능을 추가할 수 있다.

## 인터페이스 정적 메서드와 기존 유틸리티 클래스

- jdk 8 부터는 인터페이스에 정적 메서드를 선언해서 유틸리티 클래스들을 각각 만들 필요가 없어졌다.
- 그럼에도 과거 버전과 호환성을 위해 자바 API에는 Collections에서 제공하는 유틸리티 클래스같은 것들이 남아있다.

## 자바의 호환성

### 바이너리 호환성

- 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황
- 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않으므로 충족함.
- 인터페이스에 메서드를 추가했을 때는 바이너리 호환성을 유지하지만 인터페이스의 구현체 클래스를 재컴파일하면 에러가 발생한다.

### 소스 호환성

- 코드를 고쳐도 기존 프로그램을 재컴파일에 성공할 수 있음.
- 인터페이스에 메서드를 추가하면 재컴파일이 불가능하기 때문에 충족하지 않는다.

### 동작 호환성

- 코드를 바꾼 후 같은 입력값이 주어질 때 같은 동작을 한다.
- 메서드가 추가되도 기존의 메서드는 같은 동작을 하므로 충족한다.

## 추상 클래스와 인터페이스 차이

- 이제 jdk 8 부터 둘 다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다.
- 추상 클래스는 하나의 추상 클래스만 상속 가능, 인터페이스는 여러 개 구현 가능하다.
- 추상 클래스는 인스턴스 변수로 상태를 가질 수 있지만, 인터페이스는 상태를 가질 수 없다.

## 다중 상속

- 인터페이스는 다중 구현이 가능하므로 디폴트 메서드를 이용해서 다중 상속을 쓸 수 있다.
- 인터페이스의 다중 상속을 이용해서 유연한 설계가 가능
- 디폴트 메서드를 변경해도 자동으로 변경한 코드를 상속받음. (재구현 하지 않으면)

### 옳지 못한 상속

- 한 개의 메서드를 재사용하기 위해 100개의 메서드와 필드가 정의된 클래스를 상속 받는 것은 좋은 생각이 아니다.
- 델리게이션을 사용해서 지겁 호출하는 메서드를 작성하는 것이 좋다.
    - 델리게이션 : 상속이 아닌 멤버 변수로 생성해서 간접사용한다.
    - final class를 상속한 것 처럼 사용할 수 있다.
- final키워드를 이용해서 상속을 막을 수 있다.

## 해석 규칙

- 디폴트 메서드에 의해서 같은 시그니처를 갖는 메서드들의 충돌이 생길 수 있게 되었다. 컴파일러는 어떻게 충돌을 해결할까?
1. **클래스가 항상 이긴다.** 
클래스나 슈퍼 클래스에서 정의한 메서드가 디폴트 메서드보다 우선이다.
2. **서브인터페이스가 이긴다.**
상속 관계의 인터페이스에서는 서브 인터페이스가 이긴다.
3. 여전히 우선순위가 문제라면, **명시적으로 디폴트 메서드를 오버라이드** 해야한다.
- 이름이 같아도 파라미터가 다르다면 오버로딩처럼 동작한다.

### 상속 관계가 없는 인터페이스의 같은 시그니처 디폴트 메서드

- 1, 2번 규칙에 해당하지 않으므로 구별할 기준이 없다.
- 다음과 같은 컴파일 에러가 발생한다.

```java
java: types A and B are incompatible;
  class C inherits unrelated defaults for hello() from types A and B
```

- jdk 8에서 추가된 문법 : 슈퍼인터페이스 메서드 호출

```java
class C implements A, B {
    @Override
    public void hello() {
        B.super.hello(); // 슈퍼인터페이스 B
    }
}
```

## 다이아몬드 문제

- SuperInterface를 상속받는 B, C를 둘 다 받는 구현체 클래스가 있다.
- 다이어그램 모양이 다이아몬드 처럼 생겨서 다이아몬드 문제라고 불린다.

```java
interface SuperInterface {
    default void hello() {
        System.out.println("hello super interface");
    }
}

interface B extends SuperInterface{ }

interface C extends SuperInterface { }

class ImplementClass implements B, C { }

main() {
	new ImplementClass().hello();
}
```

- hello()를 호출 시 결과적으론 메서드가 하나이기 때문에 SuperInterface의 hello 가 호출된다.

### C++의 다이아몬드 문제

- 자바에서는 간단하게 해결됐지만 클래스의 다중 상속을 지원하는 C++에서는 문제가 복잡하다.
- 가장 하위 클래스는 B와 C의 객체 복사본에 접근할 수 있고, 각 클래스들은 상태도 가질 수 있으므로 상속받은 B의 메서드인지 C의 메서드인지를 명시적으로 해결해줘야 한다.

# 14장 자바 모듈 시스템

- jdk 9에서 추가된 모듈 시스템
- 궁극적으로 소프트웨어 아키텍처 즉 고수준에서는 기반 코드를 바꿔야 할 때
유추하기 쉬우므로 생산성을 높일 수 있는 소프트웨어 프로젝트가 필요하다.

### 관심사 분리 ( Separation of concerns )

- 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙
    - 서로 겹치지 않는 코드 그룹으로 분리한 모듈을 이용해 클래스 간의 관계를 시각적으로 보여줄 수 있다.
- 많은 장점을 가진다.
    - 개별 부분 재사용하기 쉬움
    - 개별 기능을 따로 작업할 수 있으므로 협업에 유리함
    - 유지보수하기 쉬워짐

### 정보 은닉

- 세부 구현을 숨기도록 장려하는 원칙
- 세부 구현을 숨김으로 코드 수정 시 다른 부분에 영향을 줄일 수 있다.
    - 코드를 관리하고 보호할 수 있다.
- 캡슐화
    - 코드의 조각이 애플리케이션의 다른 부분과 고립되어 있음.
    - 내부적인 코드 변화가 외부에 영향을 미칠 가능성을 줄여준다.
    

> 자바에선 private 키워드로 컴파일러를 이용해 캡슐화를 확인할 수 있지만 
**클래스와 패키지가 의도된 대로 공개되었는지** 컴파일러로 확인할 수 있는 기능은 없었다.
> 

## jdk 9 이전 모듈화의 한계

- 자바는 클래스, 패키지, jar 세 가지 수준의 코드 그룹화를 제공했지만, 클래스에서의 접근 제한자와 달리 패키지와 jar 수준에서는 거의 캡슐화를 지원하지 않았다.
- 한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public 으로 선언해야하고 결과적으로 모두에게 노출된다.

### Jar 지옥

- 시스템에 두 가지 버전이 다른 같은 라이브러리가 존재해도 시스템 오류로 간주되지 않는다.
- 그래서 시스템에서 버전1.0을 사용하는지 2.0을 사용하는지 지정할 수 없고 어떤 일이 일어날지 예측할 수 없다.
- 클래스의 경로에 명시적 의존성이 없어 다른 JAR에 포함된 클래스를 사용하라는 의존성을 명시적으로 알 수 없음.
    - 빠진 게 있나? 충돌이 있나? 알 수 없다.  ⇒ 물론 maven과 gradle로 거의 해결된다.

### 거대한 JDK

- jdk는 수많은 세월과 버전을 거쳐오면서 거대한 소프트웨어가 되었다.
- 자바의 낮은 캡슐화 지원 때문에 JDK의 내부에서만 사용하도록 만든 클래스인 Unsafe라는 클래스는 스프링, 네티, 모키토 등에 사용되어서 호환성을 깨지 않고 API수정이 어려워졌다.

### 모듈화의 필요성

- 이런 문제들 때문에 JDK 자체도 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기되
었다.
- 즉 JDK에서 필요한 부분만 골라 사용하고, 클래스 경로를 쉽게 유추할 수 있으며, 플랫
폼을 진화시킬 수 있는 강력한 캡슐화를 제공할 새로운 건축 구조가 필요했다

## 자바 모듈 : 큰 그림

- java 8은 모듈이라는 새로운 자바 프로그램 구조 단위를 제공한다.
- module 키워드에 이름과 바디를 추가해서 정의한다.

### 모듈 디스크립터 by java 9

- module-info.java 파일은 모듈의 이름을 지정하며 필요한 의존성(requires)과 공개 API(exports)를 정의한다.
- 모듈 디스크립터는 보통 패키지와 같은 폴더에 위치하고 한 개 이상의 패키지를 서술하고 캡슐화할 수 있지만, 단순 상황에선 패키지 중 한 개만 외부로 노출시킨다.

모듈 A는 모듈 B와 C를 필요로 하고, 패키지 모듈 B와 모듈 C를 이용해 각각 pkgB와 pkgC에 접근할 수 있다. 

모듈 C는 비슷한 방법으로 pkgD를 사용하는데 pkgD는 모듈 C에서 필요로 하지만 모듈 B에서는 pkgD를 사용할 수 없다.
![module-structure](https://github.com/devjy39/modern-java-in-action/assets/49369306/f456bc97-02cc-44d7-a04c-718a47189ec2)

### 세부적인 모듈화

- 모든 패키지가 자신의 모듈을 갖도록 설계
- 이득에 비해 설계비용이 증가한다.

### 거친 모듈화

- 한 모듈이 시스템의 모든 패키지를 포함한다.
- 모듈화의 장점을 잃음.

> 가장 좋은 방법은 시스템을 실용적으로 분해하면서 프로젝트가 이해하고 고치기 쉬운 수준으로 적절한 모듈화 정보를 주기적으로 확인하는 프로세스를 갖는 것이다.
> 

## 모듈화 애플리케이션 실행

- 보통 IDE와 빌드 시스템에서 명령이 자동으로 처리된다.

```
javac module-info.java 
 com/example/expenses/application/ExpensesApplication.java -d target

jar cvfe expenses-application.jar 
 com.example.expenses.application.ExpensesApplication -C target

//생성된 JAR(expenses-application.jar)에 어떤 폴더와 클래스 파일이 포함되어 있는지 보여준다.

added manifest
added module-info: module-info.class adding: com/(in = 0) (out= 0)(stored 0%)
adding: com/example/(in = 0) (out= 0)(stored 0%)
adding: com/example/expenses/(in = 0) (out= 0)(stored 0%)
adding: com/example/expenses/application/(in = 0) (out= 0)(stored 0%)
adding: com/example/expenses/application/ExpensesApplication.class(in = 456)
(out= 306)(deflated 32%)

// jar를 모듈화 애플리케이션으로 실행

java --module-path expenses-application.jar \
		 --module expenses/com.example.expenses.application.ExpensesApplication

--module-path : 어떤 모듈을 로드할 수 있는지 지정
--module : 실행할 메인 모듈과 클래스 지정
```

## 모듈화 사용 예제

```yaml
두 모듈 디렉터리 구조

|─ expenses.application
	|─ module-info.java            --> 패키지와 같은 수준에 존재
	|─ com
		|─ example
			|─ expenses
				|─ application
					|─ ExpensesApplication.java

|─ expenses.readers
	|─ module-info.java            --> 패키지와 같은 수준에 존재
	|─ com
		|─ example
			|─ expenses
				|─ readers
					|─ Reader.java
				|─ file
					|─ FileReader.java
				|─ http
					|─ HttpReader.java
```

### exports

- 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다.
- 기본적으로 모듈 내의 것은 캡슐화되고, 화이트리스트 기법을 이용해 강력한 캡슐화를 제공한다.

```java
module expenses.readers {
	 exports com.example.expenses.readers;
	 exports com.example.expenses.readers.file;
	 exports com.example.expenses.readers.http;
}
```

### requires

- 의존하는 모듈을 지정한다.
- 기본적으로 모든 모듈은 java.base라는 모듈에 의존한다.
    - net, io, util 등의 자바 메인 패키지를 포함한다,  —> 기본이므로 명시적으로 정의할 필요 없음.

```java
module expenses.readers {
	 requires java.base;                 // 패키지명이 아니라 모듈명

	 exports com.example.expenses.readers;       // 패키지 명
	 exports com.example.expenses.readers.file;  // 패키지 명
	 exports com.example.expenses.readers.http;  // 패키지 명
}
```

> 이로써 클래스의 가시성을 더 잘 제어할 수 있게되었다.
- 제한된 클래스만 공개 가능. 
- 한 모듈의 내에서만 공개 가능.
> 

### requires transitive

- 의존하는 모듈을 전이시킬 수 있다.

```java
module com.iteratrlearning.ui {
	 requires transitive com.iteratrlearning.core; // ui 모듈을 의존하면 core까지 사용 가능
	 exports com.iteratrlearning.ui.panels;
	 exports com.iteratrlearning.ui.widgets;
}

module com.iteratrlearning.application { // core 모듈도 접근 가능해진다.
	 requires com.iteratrlearning.ui;
}
```

### export to

- 모듈을 공개할 사용자를 지정하여 제한할 수 있다.

```java
module my.module {
    // 패키지 com.example.mypackage를 다른 모듈에 공개합니다.
    exports com.example.mypackage to other.module1, other.module2;
}
```

### open, opens

- 모든 패키지를 오픈 가능

```java
open module my.module {
    // 모든 패키지를 오픈 
}
```

- 자바9 부터는 리플렉션으로 객체의 비공개 상태를 확인할 수 없기 때문에 공개하려면 명시적으로 open 키워드를 사용해야 한다.

### 모듈의 이름 규칙

- 오라클은 패키지명처럼 인터넷 도메인명을 역순으로 모듈의 이름을 정하도록 권고한다.
- 모듈명은 노출된 주요 API 패키지와 이름이 같아야 한다.

## 모듈 컴파일과 패키징

- 각 모듈은 독립적으로 컴파일되므로 자체적으로 각각이 한 개의 프로젝트다.

### 자동 모듈

- 모듈 경로상에 있으나 module-info 파일을 가지지 않은 모든 JAR는 자동 모듈이 된다.
자동 모듈은 암묵적으로 자신의 모든 패키지를 노출시킨다.

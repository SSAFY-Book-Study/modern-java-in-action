# 챕터4

## 11.1 값이 없는 상황을 어떻게 처리할까?

```java
public class Person {
 private Car car;
 public Car getCar() { return car; }
}
public class Car {
 private Insurance insurance;
 public Insurance getInsurance() { return insurance; }
}
public class Insurance {
 private String name;
 public String getName() { return name; }
}
```

## 11.1.1 보수적인 자세로 NullPointerException 줄이기

```java
public String getCarInsuranceName(Person person) {
 if (person != null) {
 Car car = person.getCar();
 if (car != null) {
 Insurance insurance = car.getInsurance();
 if (insurance != null) {
 return insurance.getName();
 }
 }
 }
 return "Unknown";
}
```

## 11.1.2 null 때문에 발생하는 문제

- 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
- 코드를 어지럽힌다 : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코
드 가독성이 떨어진다.
- 아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이
없음을 표현하는 방법으로는 적절하지 않다.
- 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는
데 그것이 바로 null 포인터다.
- 형식 시스템에 구멍을 만든다 : null은 무형식이며 정보를 포함하고 있지 않으므로 모든
참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의
다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

## 11.1.3 다른 언어는 null 대신 무얼 사용하나?

```java
def carInsuranceName = person?.car?.insurance?.name
```

- 그루비는 안전 내비게이션 연산자를 도입해서 null 문제를 해결했다.
- 자바 8은 선택형값 개념의 영향을 받아서 java.util.Optional<T>라는 새로운 클래스를 제공한다.

## 11.2 Optional 클래스 소개

- Optional은 선택형값을 캡슐화하는 클래스다.
    - 값이 있으면 Optional 클래스는 값을 감싼다.
    - 없으면 Optional.empty 메서드로 Optional을 반환한다.

```java
public class Person {
 private Optional<Car> car;
 public Optional<Car> getCar() {
 return car; }

 }
public class Car {
 private Optional<Insurance> insurance;
 public Optional<Insurance> getInsurance() {
 return insurance;
 }
}
public class Insurance {
 private String name;
 public String getName() {
 return name;
 }
}
```

## 11.3 Optional 적용 패턴

- 지금까지 Optional 형식을 이용해서 우리 도메인 모델의 의미를 더 명확하게 만들 수 있었으며 null 참조 대신 값이 없는 상황을 표현할 수 있음을 확인했다

## 11.3.1 Optional 객체 만들기

### 빈 Optional

```java
Optional<Car> optCar = Optional.empty();
```

### null이 아닌 값으로 Optional 만들기

```java
Optional<Car> optCar = Optional.of(car);
```

### null값으로 Optional 만들기

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

## 11.3.2 맵으로 Optional의 값을 추출하고 변환하기

```java
String name = null;
if(insurance != null) {
 name = insurance.getName();
}
/////////////////////
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
 Optional<String> name = optInsurance.map(Insurance::getName);
```

## 11.3.3 flatMap으로 Optional 객체 연결

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name =
 optPerson.map(Person::getCar)
 .map(Car::getInsurance)
 .map(Insurance::getName);
```

- 중첩 Optional 객체 구조로 컴파일 되지 않음
- flatMap 메서드 덕분에 이차원 Optional이 하나의 Optional로 바뀐다.

```java
public String getCarInsuranceName(Optional<Person> person) {
 return person.flatMap(Person::getCar)
 .flatMap(Car::getInsurance)
 .map(Insurance::getName)
 .orElse("Unknown");
}
```

### Optional을 이용한 Person/Car/Insurance 참조 체인

- 평준화 과정이란 이론적으로 두 Optional을 합치는 기능을 수행하면서 둘 중 하나라도 null이면 빈 Optional을 생성하는 연산이다.
- 호출 체인 중 어떤 메서드가 빈 Optional을 반환한다면 전체 결과로 빈 Optional을 반환하고
아니면 관련 보험회사의 이름을 포함하는 Optional을 반환한다.

### 도메인 모델에 Optional을 사용했을 떄 데이터 직렬화 할 수 없는 이유

- Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 직렬화 인터페이스를 구현하지 않는다.
- 도메인 모델에 Optional을 사용한다면 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다.

```java
public class Person {
 private Car car;
 public Optional<Car> getCarAsOptional() {
 return Optional.ofNullable(car);
 }
}
```

## 11.3.4 Optional 스트림 조작

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
 return persons.stream()
 .map(Person::getCar)
 .map(optCar -> optCar.flatMap(Car::getInsurance))
 .map(optIns -> optIns.map(Insurance::getName))
 .flatMap(Optional::stream)
 .collect(toSet()); 
///////////////
Stream<Optional<String>> stream = ...
Set<String> result = stream.filter(Optional::isPresent)
 .map(Optional::get)
 .collect(toSet());
```

## 11.3.5 디폴트 액션과 Optional 언랩

- get()은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드다. 메서
드 get은 래핑된 값이 있으면 해당 값을 반환하고 값이 없으면 NoSuchElementException
을 발생시킨다. 따라서 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니면
get 메서드를 사용하지 않는 것이 바람직하다. 결국 이 상황은 중첩된 null 확인 코드를
넣는 상황과 크게 다르지 않다.
- orElse 메서드를 이용하면 Optional 이 값을 포함하지 않을 때 기본값을 제공할 수 있다.
- orElseGet(Supplier<? extends T> other)는 orElse 메서드에 대응하는 게으른 버전의
메서드다. Optional에 값이 없을 때만 Supplier가 실행되기 때문이다. 디폴트 메서드를
만드는 데 시간이 걸리거나(효율성 때문에) Optional이 비어있을 때만 기본값을 생성하
고 싶다면(기본값이 반드시 필요한 상황) orElseGet(Supplier<? extends T> other)를
사용해야 한다.
- orElseThrow(Supplier<? extends X> exceptionSupplier)는 Optional이 비어있을 때
예외를 발생시킨다는 점에서 get 메서드와 비슷하다. 하지만 이 메서드는 발생시킬 예외
의 종류를 선택할 수 있다.
- ifPresent(Consumer<? super T> consumer)를 이용하면 값이 존재할 때 인수로 넘겨준
동작을 실행할 수 있다. 값이 없으면 아무 일도 일어나지 않는다.
자바 9에서는 다음의 인스턴스 메서드가 추가되었다.
- ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction). 이 메서드
는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent
와 다르다.

## 11.3.6 두 Optional 합치기

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
 Optional<Person> person, Optional<Car> car) {
 if (person.isPresent() && car.isPresent()) {
 return Optional.of(findCheapestInsurance(person.get(), car.get()));
 } else {
 return Optional.empty();
 }
}
```

- person과 car의 시그니처만으로 둘다 아무 값도 반환하지 않을 수 있따는 정보를 명시적으로 보여줌

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
 Optional<Person> person, Optional<Car> car) {
 return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

- 첫 번째 Optional에 flatMap을 호출했으므로 첫 번째 Optional이 비어 있다면 전달한 람
다 표현식이 실행되지 않고 그대로 빈 Optional을 반환한다.
- 반면 person값이 있으면 flatMap 메서드에 필요한 Optional<Insurance>를 반환하는 Function의 입력으로 person을 사용한다.
- 이 함수의 바디에서는 두 번째 Optional에 map을 호출하므로 Optional이 car값을 포함하지 않으면 Function은 빈 Optional을 반환하므로 결국 nullSafeFindCheapestInsurance는 빈 Optional을 반환한다.
- 마지막으로 person과 car가 모두 존재하면 map 메서드로 전달한 람다 표현식이 findCheapestInsurance 메서드를 안전하게 호출할 수 있다.

## 11.3.7 필터로 특정값 거르기

```java
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){ 
 System.out.println("ok");
}

Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
 "CambridgeInsurance".equals(insurance.getName()))
 .ifPresent(x -> System.out.println("ok"));
```

- filter 메서드는 프레디케이트를 인수로 받는다. Optional 객체가 값을 가지며 프레디케이트
와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다.

## 11.4 Optional을 사용한 실용 예제

- 지금까지 배운 것처럼 새 Optional 클래스를 효과적으로 이용하려면 잠재적으로 존재하지 않는 값의 처리 방법을 바꿔야 한다. 즉, 코드 구현만 바꾸는 것이 아니라 네이티브 자바 API와 상호작용하는 방식도 바꿔야 한다.

## 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

```java
//HashMap 사용시
Object value = map.get("key");

Optional<Object> value = Optional.ofNullable(map.get("key"));
```

- if-then-else
- Optional.ofNullable

## 11.4.2 예외와 Optional클래스

```java
public static Optional<Integer> stringToInt(String s) {
 try {
 return Optional.of(Integer.parseInt(s));
 } catch (NumberFormatException e) {
 return Optional.empty();
 }
}
```

## 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유

- Optional의 최대 요소 수는 한 개이므로 Optional에서는 기본형 특화 클래스로 성능을 개선할 수 없다.

## 12.1 LocalDate,LocalTime,Instant,Duration,Period클래스

- 먼저 간단한 날짜와 시간 간격을 정의해보자. java.time 패키지는 LocalDate, LocalTime,
LocalDateTime, Instant, Duration, Period 등 새로운 클래스를 제공한다

## 12.1.1 LocalDate와 LocalTime 사용

```java
LocalDate date = LocalDate.of(2017, 9, 21); 2017-09-21
int year = date.getYear(); 2017
Month month = date.getMonth(); SEPTEMBER
int day = date.getDayOfMonth(); 21
DayOfWeek dow = date.getDayOfWeek(); THURSDAY
int len = date.lengthOfMonth(); 31 (3월의 일 수)
boolean leap = date.isLeapYear();
```

- TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스다.
- 열거자 ChronoField는 TemporalField 인터페이스를 정의하므로 다음 코드에서 보여주는 것처럼 ChronoField의 열거자 요소를 이용해서 원하는 정보를 쉽게 얻을 수 있다.

```java
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);

LocalTime time = LocalTime.of(13, 45, 20); 13:45:20
int hour = time.getHour(); 13
int minute = time.getMinute(); 45
int second = time.getSecond();

LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

## 12.1.2 날짜와 시간 조합

```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

## 12.1.3 Instant 클래스 : 기계의 날짜와 시간

- LocalDate 등을 포함하여 사람이 읽을 수 있는 날짜 시간 클래스에서 그랬던 것처럼 Instant
클래스도 사람이 확인할 수 있도록 시간을 표시해주는 정적 팩토리 메서드 now를 제공한다.
- 하지만 Instant는 기계 전용의 유틸리티라는 점을 기억하자. 즉, Instant는 초와 나노초 정보를
포함한다. 따라서 Instant는 사람이 읽을 수 있는 시간 정보를 제공하지 않는다. 예를 들어 다
음 코드를 보자.

```java
int day = Instant.now().get(ChronoField.DAY_OF_MONTH);
java.time.temporal.UnsupportedTemporalTypeException: Unsupported field: DayOfMonth
```

## 12.1.4 Duration과 Period 정의

- LocalDateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어진 클래스로 두 인
스턴스는 서로 혼합할 수 없다.
- 또한 Duration 클래스는 초와 나노초로 시간 단위를 표현하므로 between 메서드에 LocalDate를 전달할 수 없다.

## 12.2 날짜 조정, 파싱, 포매팅

```java
LocalDate date1 = LocalDate.of(2017, 9, 21); 2017-09-21
LocalDate date2 = date1.plusWeeks(1); 2017-09-28
LocalDate date3 = date2.minusYears(6); 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); 2012-03-28
```

## 12.2.1 TemporalAdjusters 사용하기

```java
import static java.time.temporal.TemporalAdjusters.*;
LocalDate date1 = LocalDate.of(2014, 3, 18); 2014-03-18
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); 2014-03-31
```

## 12.2.2 날짜와 시간 객체 출력과 파싱

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); 20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); 2014-03-18

LocalDate date1 = LocalDate.parse("20140318",DateTimeFormatter.BASIC_ISO_DATE); 
LocalDate date2 = LocalDate.parse("2014-03-18",DateTimeFormatter.ISO_LOCAL_DATE);
```

## 12.3 다양한 시간대와 캘린더 활용 방법

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone);
```

## 13.1 변화하는 API

## 13.1.1 API 버전 1

```java
public interface Resizable extends Drawable {
 int getWidth();
 int getHeight();
 void setWidth(int width);
 void setHeight(int height);
 void setAbsoluteSize(int width, int height);
}

public class Ellipse implements Resizable {
 ...
}
```

## 13.1.2 API 버전 2

```java
public interface Resizable {
 int getWidth();
 int getHeight();
 void setWidth(int width);
 void setHeight(int height);
 void setAbsoluteSize(int width, int height);
 void setRelativeSize(int wFactor, int hFactor); API 버전 2에 추가된 새로운 메서드
}
```

### 사용자가 겪는 문제

- Resizable을 고치면 몇 가지 문제가 발생한다.
    - 첫 번째로 Resizable을 구현하는 모든 클래스는 setRelativeSize 메서드를 구현해야 한다.
    - 하지만 라이브러리 사용자가 직접 구현한 Ellipse는 setRelativeSize 메서드를 구현하지 않는다.
    - 인터페이스에 새로운 메서드를 추가하면 바이너리 호환성은 유지된다.
        - 바이너리 호환성이란 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스 파일 구현이 잘 동작한다는 의미다.
        - 하지만 언젠가는 누군가가 Resizable을 인수로 받는 Utils.paint에서 setRelativeSize를 사용하도록코드를 바꿀 수 있다. 이때 Ellipse 객체가 인수로 전달되면 Ellipse는setRelativeSize 메서드를 정의하지 않았으므로 런타임에 다음과 같은 에러가 발생할 것이다.
        

디폴트 메서드로 이 모든 문제를 해결할 수 있다. 디폴트 메서드를 이용해서 API를 바꾸면 새
롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다.

## 13.2 디폴트 메서드란 무엇인가?

- 자바 8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 디폴트 메서드default method를 제공한다.
- 이제 인터페이스는 자신을 구현하는 클래스에서 메서드를 구현하지 않을 수 있는 새
로운 메서드 시그니처를 제공한다.
- 그럼 디폴트 메서드는 누가 구현할까?
    - 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공한다(그래서 이를 디폴트 메서드라고 부른다).

## 13.3 디폴트 메서드 활용 패턴

- 선택형 메서드
- 동작 다중 상속

## 13.3.1 선택형 메서드

```java
interface Iterator<T> {
 boolean hasNext();
 T next();
 default void remove() {
 throw new UnsupportedOperationException();
 }
}
```

- 기본 구현이 제공되므로 Iterator 인터페이스를 구현하는 클래스는 빈 remove 메서드를 구현
할 필요가 없어졌고, 불필요한 코드를 줄일 수 있다.

## 13.3.2 동작 다중 상속

- 디폴트 메서드를 이용하면 기존에는 불가능했던 동작 다중 상속 기능도 구현할 수 있다

### 다중 상속 형식

- 결과적으로 ArrayList는 AbstractList, List, RandomAccess, Cloneable, Serializable, Iterable,
Collection의 서브형식subtype이 된다.

### 기능이 중복되지 않는 최소의 인터페이스

```java
public interface Rotatable {
 void setRotationAngle(int angleInDegrees);
 int getRotationAngle();
 default void rotateBy(int angleInDegrees) { rotateBy 메서드의 기본 구현
 setRotationAngle((getRotationAngle () + angleInDegrees) % 360);
 }
}
```

## 13.4 해석 규칙

- 자바의 클래스는 하나의 부모 클래스만 상속받을 수 있지만 여러 인터페이스를 동시에 구현할 수 있다.

```java
public interface A {
 default void hello() {
 System.out.println("Hello from A");
 }
}
public interface B extends A {
 default void hello() {
 System.out.println("Hello from B");
 }
}
public class C implements B, A {
 public static void main(String... args) {
 new C().hello(); 무엇이 출력될까?
 }
}
```

## 13.4.1 알아야 할 세 가지 해결 규칙**

1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우
선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에
서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. 즉, B가 A를
상속받는다면 B가 A를 이긴다.
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클
래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.

## 13.4.2 디폴트 메서드를 제공하는 서브인터페이스가 이긴다.

```java
public class D implements A{ }
public class C extends D implements B, A {
 public static void main(String... args) {
 new C().hello(); 무엇이 출력될까?
 }
}
```

- 1번 규칙은 클래스의 메서드 구현이 이긴다고 설명한다. D는 hello를 오버라이드하지 않았고
단순히 인터페이스 A를 구현했다.

## 13.4.3 충돌 그리고 명시적인 문제 해결

```java
public interface A {
 default void hello() {
 System.out.println("Hello from A");
 }
}
public interface B {
 default void hello() {
 System.out.println("Hello from B");
 }
}
public class C implements B, A { }

“Error: class C inherits unrelated defaults for hello( ) from types B and A.”
```

- 즉, 클래스 C에서 hello 메서드를 오버라이드한 다음에 호출하려는 메서드를 명시적으로 선택해야 한다.

## 13.4.4 다이아몬드 문제

```java
public interface A {
 default void hello(){
 System.out.println("Hello from A");
 }
}
public interface B extends A { }
public interface C extends A { }
public class D implements B, C {
 public static void main(String... args) {
 new D().hello(); 무엇이 출력될까?
 }
}
```

## 14.1 압력 : 소프트웨어 유추

자바 모듈 시스템을 자세히 살펴보기 전에 어떤 동기와 배경으로 자바 언어 설계자들이 목표를
정했는지 이해해보자

- 소프트웨어 아키텍처 즉 고수준에서는 기반 코드를 바꿔야 할 때 유추하기 쉬우므로 생산성을 높일 수 있는 소프트웨어 프로젝트가 필요하다.
- 관심사 분리
- 정보 은닉

## 14.1.1 관심사 분리

- 관심사분리 SoC, Separation of concerns는 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다.
- 다양한 형식으로 구성된 지출을 파싱하고, 분석한 다음 결과를 고객에게 요약 보고
하는 회계 애플리케이션을 개발한다고 가정하자.

SoC 원칙은 모델, 뷰, 컨트롤러 같은 아키텍처 관점 그리고 복구 기법을 비즈니스 로직과 분리
하는 등의 하위 수준 접근 등의 상황에 유용하다. SoC 원칙은 다음과 같은 장점을 제공한다.

● 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다.
● 개별 부분을 재사용하기 쉽다.
● 전체 시스템을 쉽게 유지보수할 수 있다.

## 14.1.2 정보 은닉

- 정보 은닉은 세부 구현을 숨기도록 장려하는 원칙이다.
- 캡슐화는 특정 코드 조각이 애플리케이션 의 다른 부분과 고립되어 있음을 의미한다.
- 캡슐화된 코드의 내부적인 변화가 의도치 않게 외부에 영향을 미칠 가능성이 줄어든다.

클래스와 패키지가 의도된 대로 공개되었는지를 컴파일러로 확인할 수 있는 기능
이 없었다.

## 14.1.3 자바 소프트웨어

- 클래스, 인터페이스를 그룹으로 만들어 코드를 그룹화할 수 있다.
- 코드 자체를 보고 소프트웨어의 동작을 추론하긴 현실적으로 어렵다.
- 따라서 UML 다이어그램같은 도구를 이용하면 그룹 코드 간의 의존성을 시각적으로 보여줄 수 있으므로 소프트웨어를 추론하는데 도움이 된다.

모듈화의 장점을 살펴봤는데 자바의 모듈 지원이 어떤 변화를 가져왔는지 궁금할 것이다.

## 14.2 자바 모듈 시스템을 설계한 이유

- 자바 언어와 컴파일러에 새로운 모듈 시스템이 추가된 이유를 설명한다.
- 먼저 자바 9 이전의 모듈화 한계를 살펴본다. 그리고 JDK 라이브러리와 관련한 배경 지식을 제공하고 모듈화가 왜 중요한지 설명한다.

## 14.2.1 모듈화의 한계

- 자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는 데 한계가 있었다.
- 자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공한다.
- 클래스와 관련해 자바는 접근 제한자와 캡슐화를 지원했다.
- 하지만 패키지와 JAR 수준에서는 캡슐화를 거의 지원하지 않았다.

### 제한된 가시성 제어

- public, protected,패키지 수준, private 이렇게 네 가지 가시성 접근자가 있다.

### 클래스 경로

- 애플리케이션을 번들하고 실행하는 기능과 관련해 자바는 태생적으로 약점을 갖고 있다.
- 클래스를 모두 컴파일한 다음 보통 한 개의 평범한 JAR 파일에 넣고 클래스 경로에 이 JAR 파일을 추가해 사용할 수 있다. 그러면 JVM이 동적으로 클래스 경로에 정의된 클래스를 필요할 때 읽는다.

안타깝게도 클래스 경로와 JAR 조합에는 몇 가지 약점이 존재한다.

- 첫째, 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다. 예를 들어 파싱 라이브러리의 JSONParser 클래스를 지정할 때 버전 1.0을 사용하는지 버전 2.0을 사용하는지 지정할 수가 없으므로 클래스 경로에 두 가지 버전의 같은 라이브러리가 존재할 때 어떤 일이 일어날지 예측할 수 없다. 다양한 컴포넌트가 같은 라이브러리의 다른 버전을 사용하는 상황이 발생할 수 있는 큰 애플리케이션에서 이런 문제가 두드러진다.
- 둘째, 클래스 경로는 명시적인 의존성을 지원하지 않는다. 각각의 JAR 안에 있는 모든 클래스
는 classes라는 한 주머니로 합쳐진다. 즉 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용하
라고 명시적으로 의존성을 정의하는 기능을 제공하지 않는다. 이 상황에서는 클래스 경로 때문에 어떤 일이 일어나는지 파악하기 어려우며 , 다음과 같은 의문이 든다.
    - 빠진 게 있는가?
    - 충돌이 있는가?

## 14.2.2 거대한 JDK

- 자바 개발 키트(JDK)는 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합이다.
- 가장 익숙한 도구로 자바 프로그램을 컴파일하는 javac, 자바 애플리케이션을 로드하고 실행
하는 java, 입출력을 포함해 런타임 지원을 제공하는 JDK 라이브러리, 컬렉션, 스트림 등이 있
다

## 14.2.3 OSGi와 비교

- OSGi의 각 번들이 자체적인 클래스 로더를 갖는 반면, 자바 9 모듈 시스템의 직소
는 애플리케이션당 한 개의 클래스를 사용하므로 버전 제어를 지원하지 않는다.

## 14.3 자바 모듈 : 큰 그림

- 모듈 디스크립터는 보통 패키지와 같은 폴더에 위치하며 한 개 이상의 패키지를 서술하고 캡슐화할 수 있지만 단순한 상황에서는 이들 패키지 중 한 개만 외부로 노출시킨다.

## 14.4 자바 모듈 시스템으로 애플리케이션 개발하기

- 작은 모듈화 애플리케이션을 구조화하고, 패키지하고, 실행하는 방법을
배운다.

## 14.4.1 애플리케이션 셋업

- 프로젝트가 점점 커지면서 많은 내부 구현이 추가되면 이때부터 캡슐화와 추론의 장점이 두드러진다

## 14.4.2 세부적인 모듈화와 거친 모듈화

- 모듈화는 소프트웨어 부식의 적이다.

## 14.4.3 자바 모듈 시스템 기초

- 자바 9 모듈 시스템에서 버전 선택 문제를 크게 고려하지 않았고 따라서 버전 기능은 지원하지 않는다. 대신 버전 문제는 빌드 도구나 컨테이너 애플리케이션에서 해결해야 할 문제로 넘겼다

## 14.5 여러 모듈 활용하기

- expenses.application와 expenses. readers 두 모듈간의 상호 작용은 자바 9에서 지정한 export, requires를 이용해 이루어진다.

## 14.5.1 exports 구문

- exports라는 구문이 새로 등장했는데 exports는 다른 모듈에서 사용할 수 있도록 특정 패키지
를 공개 형식으로 만든다. 기본적으로 모듈 내의 모든 것은 캡슐화된다.

## 14.5.2 requires 구분

- requires라는 구분이 새로 등장했는데 requires는 의존하고 있는 모듈을 지정한다.
- 기본적으로 모든 모듈은 java.base라는 플랫폼 모듈에 의존하는데 이 플랫폼 모듈은 net, io, util 등의 자바 메인 패키지를 포함한다.

## 14.8 모듈 정의와 구문들

- 자바 모듈 시스템은 커다란 짐승이다

## 14.8.1 requires

- requires 구문은 컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다. 예를 들
어 com.iteratrlearning.application은 com.iteratrlearning.ui 모듈에 의존한다.

```java
module com.iteratrlearning.application {
requires com.iteratrlearning.ui;
}
```

- 그러면 com.iteratrlearning.ui에서 외부로 노출한 공개 형식을 com.iteratrlearning.
application에서 사용할 수 있다.

## 14.8.2 exports

```java
module com.iteratrlearning.ui {
requires com.iteratrlearning.core;
exports com.iteratrlearning.ui.panels;
exports com.iteratrlearning.ui.widgets;
}
```

exports 구문은 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다. 아
무 패키지도 공개하지 않는 것이 기본 설정이다. 어떤 패키지를 공개할 것인지를 명시적으로
지정함으로 캡슐화를 높일 수 있다. 다음 예제에서는 com.iteratrlearning.ui.panels와 com.
iteratrlearning.ui.widgets를 공개했다(참고로 문법이 비슷함에도 불구하고 exports는 패
키지명을 인수로 받지만 requires는 모듈명을 인수로 받는다는 사실에 주의하자).

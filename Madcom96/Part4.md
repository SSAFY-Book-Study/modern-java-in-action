# null 대신 Optional 클래스
## 1. 값이 없는 상황 처리
- null 값을 매번 검사해주는 코드를 작성할 수 있다.
	- 코드의 구조가 망가지고 가독성이 떨어진다.

- null 때문에 발생하는 문제
	- 에러의 근원이다.
	- 코드를 어지럽힌다.
	- 아무 의미가 없다.
	- 자바 철학에 위배된다. (포인터를 숨겼으나 null포인터는 제외)
	- 형식 시스템에 구멍을 만든다.

## 2. Optional 클래스
- 값이 있으면 클래스는 값을 감싼다.
- 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.
	- 모델의 의미semantic가 더 명확해진다.
	- 모든 null 참조를 Optional로 대치하는 것은 바람직하진 않다.
		- 이름 그대로 선택형임을 나타내, 이해하기 더 쉬운 API를 설계하도록 돕는다.

## 3. Optional 적용 패턴
```java
Optional<Car> optCar = Optional.empty(); 
// 빈 Optional객체
Optional<Car> optCar = Optional.of(car); 
// car라는 객체를 넣은 Optional
Optional<car> optCar = Optional.ofNullable(car);
// null 값을 저장할 수 있는 Optional
// car이 null이면 빈 Optional 객체
```

- 맵으로 Optional의 값을 추출하고 변환하기
```java
Optional<Insurance>optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsureance.map(Insurance::getName);
```
- flatMap으로 Optional객체 연결
```java
person.flatMap(Person::getCar)
		.flatMap(Car::getInsurance)
		.map(Insurance::getName)
		.orElse("Unknow"); // 결과 Optional이 비어있으면 기본값 사용
```
- 2차원 이상의 Optional도 map을 할 수 있게 해준다.

#### Optional 스트림 조작
```java
Stream<Optional<String>> stream ...
Set<String> result = stream.filter(Optional::isPresent)
							.map(Optional::get)
							.collect(toSet());
```

#### 디폴트 액션과 Optional 언랙
- get()은 값을 읽는 가장 간단하고 가장 안전하지 않은 메서드
- orElse(T other)를 이용하면 Optional이 값을 포함하지 않을 때 기본값을 제공
- orElseGet(Supplier)는 orElse에 대응하는 게으른 버전 메서드
	- Optional에 값이 없을때만 Supplier가 실행
 - orElseThrow(Supplier)는 값이 없을때 예외를 발생. 예외의 종류 선택가능
 - isPresent(Consumer)를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행
 - ifPresentOrElse(Consumer, Runnable) 자바9에서 추가. 비었을 때 실행할 Runnable을 받는다.
 
#### Optional 언랩하지 않고 두 Optional 합치기
```java
Optional<Insurance> nullSafeFindCheapest(Optional<Person> person, Optional<Car> car) {
	return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
// 조건문을 사용하지 않고 합칠수가 있다.
```

#### 필터로 특정값 거르기
```java
person.filter(p -> p.getAge() >= minAge)
		.flatMap(Person::getCar)
		.flatMap(Car::getInsurance)
		.map(Insurance::getName)
		.orElse("Unknow");
```

#### 기본형 Optional을 사용하지 말아야 하는 이유
- 기본형 특화 OptionalInt, OptionalLong, OptionalDouble등의 클래스를 제공한다.
- 하지만 Stream과 다르게 Optional의 최대 요소 수는 한 개이므로 Optional에서는 기본형 특화 클래스로 성능을 개선할 수 없다.
- map, flatMap, filter 등을 지원하지도 않는다.

# 새로운 날짜와 시간 API
### 필요 이유
- 기존의 직관적이지 않은 Date 클래스
- 대안으로 나온 Calendar클래스 또한 쉽게 에러를 일으키는 설계문제를 가짐
- DateFormat은 스레드에 안전하지 않은 문제.

## 1. LocalDate, LocalTime, Instant, Duration, Period 클래스
#### LocalDate, LocalTime
```java
LocalDate date = LocalDate.of(2017, 9, 21);
int year = date.getYear();  
Month month = date.getMonth(); // SEPTEMBER
int day = date.getDayOfMonth();
DayOfWeek dow = date.getDayOfWeek(); // THURSDAY
int len = date.lengthOfMonth(); // 해당 달의 일 수
boolean leap = date.isLeapYear(); // 윤년인가? false, true
```
- `LocalDate date = LocalDate.of(2017, 9, 21);`
	- 2017. 9. 21을 저장하는 LocalDate 인스턴스 생성
- `LocalDate today = LocalDate.now();` 
	- 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.
- `date.get(ChronoField.YEAR);`
	- `date.getYear();` 내장메서드도 사용 가능

```java
LocalTime time = LocalTime.of(13, 45, 20);
int hour = time.getHour(); // 13 
int minute = time.getMinute(); // 45 
int second = time.getSecond(); // 20
```
- 위와 같이 LocalTime 객체 생성 가능

```java
LocalDate.parse("2017-09-21"); // LocalDate 객체 반환
LocalTime.parse("13:45:20"); // LocalTime 객체 반환
```

#### 날짜와 시간 조합
```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

#### Instant 클래스 : 기계의 날짜와 시간
- Instant 클래스의 ofEpochSecond 메서드의 두 번째 인수를 이용해 나노초 단위로 시간을 보정할 수 있다.
- `Instant.ofEpochSecond(2, 1_000_000_000)` 2 초 이후의 1억 나노초 (1초)
- 기계 전용의 유틸리티이므로 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.

#### Duration과 Period 정의
- Duration 초 & 나노초
	- between 정적 팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다.
- Period 년월일
	- between 팩토리 메서드를 이용하면 두 LocalDate의 차이를 확인할 수 있다.
```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);  
Period threeWeeks = Period.ofWeeks(3);  
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1);
// 위와 같이 객체를 사용하지 않고 클래스를 만들 수 있는 다양한 팩토리 메서드도 존재
```

#### 날짜 조정, 파싱, 포매팅 : withAttribute
```java
LocalDate date1 = LocalDate.of(2017, 9, 21);  
LocalDate date2 = date1.withYear(2011);  
LocalDate date3 = date2.withDayOfMonth(25);  
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2);
// 기존 객체를 바꾸지 않고 새로운 객체 반환
```

#### TemporalAdjusters
```java
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
// 그 날짜의 일요일 날짜 객체 반환
```

#### 날짜와 시간 객체 출력과 파싱
```java
LocalDate date = LocalDate.of(2014, 3, 18);  
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318 
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);
// 2014-03-18
```
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy"); 
// 포맷 지정 가능
LocalDate date1 = LocalDate.of(2014, 3, 18);  
String formattedDate = date1.format(formatter);  
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

#### 대안 캘린더 시스템 사용하기
```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18); JapaneseDate japaneseDate = JapaneseDate.from(date);
// 위의 JapaneseDate와 같이
// ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate
// 등을 지원한다.
```

# 디폴트 메서드
- 인터페이스에 대한 수정이 발생하면 모든 구현체에 새로 구현을 해 줘야 하는 문제 발생
- 해당 문제를 해결하는 자바 8의 기능
	1. 정적 메서드
	2. 디폴트 메서드

## 1. 변화하는 API
- Resizable 이라는 인터페이스 생성
- Resizable 수정. setRelativeSize 메서드 생성
- 이제 Resizable을 구현하는 모든 클래스는 setRelativeSize 메서드를 구현해야 한다.
	- 하지만 사용자가 직접 구현한 클래스는 setRelativeSize메서드를 구현하지 않는다면
	- 이때 `바이너리 호환성`은 유지된다.
	- 직접 구현한 클래스를 통해 어디선가 setRelativeSize를 호출하면 그때 런타임 에러 발생
 - 사용자가 Resizable을 구현하는 클래스를 포함하는 전체 애플리케이션을 재빌드할 때 컴파일 에러 발생
 > 바이너리 호환성 : 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황
 소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음
 동작 호환성 : 코드를 바꾼 다음에도 같은 입력값에 대해 같은 동작 실행

#### 디폴트 메서드
- 디폴트 메서드를 통해 위의 문제를 해결할 수 있다.
- 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공한다.
```java
public interface Sized { int size();
	default boolean isEmpty() { return size() == 0;}
} // default 키워드를 추가해준다.
```

#### 디폴트 메서드 활용 패턴
```java
interface Iterator<T> { 
	boolean hasNext();  
	T next();  
	default void remove() {
		throw new UnsupportedOperationException();
	}
} // 디폴트 메서드 추가하기

public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable { 
// 네개의인터페이스를구현한다.
} // 동작 다중 상속
```

#### 해석 규칙
1. 클래스가 항상 이긴다.
2. 1번 규칙 이외의 상황에서는 서브 인터페이스가 이긴다.
3. 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.
	- A, B 인터페이스를 구현한다면 B.super.hello()와 같이 명시적으로 해결

# 자바 모듈 시스템
- 자바 9에서 모듈 시스템 지원
## 1. 소프트웨어 유추
- 관심사 분리 SoC, Separation of Concerns
	- 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙
	- 예시로 회계 프로그램을 만든다면, 파싱, 분석, 레포트 기능을 겹치지 않는 코드 그룹으로 분리할 수 있다.
	 - **클래스를 그룹화한 모듈을 이용해 애플리케이션의 클래스 간의 관계를 시각적으로 보여줄 수 있다.**
- 정보 은닉
	- 세부 구현을 숨기도록 장려
	- 자바 9 이전까지는 **클래스와 패키지가 의도된 대로 공개되었는지**를 컴파일러로 확인할 수 있는 기능이 없었다.

## 2. 자바 모듈 시스템을 설계한 이유
- 제한된 가시성 제어: public, protected, 패키지 수준, private
	- 한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 이들을 선언해야 한다.
	- 보안 측면에서 코드가 노출된다.
 
#### 클래스 경로
- 클래스를 모두 컴파일한 다음 보통 한 개의 평범한 JAR 파일에 넣고 클래스 경로에서 이 JAR 파일을 추가해 사용할 수 있다.
- 약점
	- 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다.
	- 클래스 경로는 명시적인 의존성을 지원하지 않는다.
		- 각각의 JAR 안에 있는 모든 클래스는 classes라는 한 주머니로 합쳐진다.
		- 빠진 것이 있는지, 충돌이 있는지 확인이 어렵다.
  
#### 모듈 정의와 구문들
```java
module com.iteratrlearning.application {
	requires com.iteratrlearning.ui;
}

module com.iteratrlearning.ui {
   requires com.iteratrlearning.core;
   exports com.iteratrlearning.ui.panels;
   exports com.iteratrlearning.ui.widgets;
}
```
- requires 구문은 컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다.
	- 모듈명을 인수로 받는다.
- exports 구문은 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다.
	- 패키지 명을 인수로 받는다.
 
```java
module com.iteratrlearning.ui {  
	requires transitive com.iteratrlearning.core;

	exports com.iteratrlearning.ui.panels;
	exports com.iteratrlearning.ui.widgets;
}

module com.iteratrlearning.application {
	requires com.iteratrlearning.ui;
}
```
- requires transitive를 사용
- 결과적으로 com.iteratrlearning.application 모듈은 com.iteratrlearning.core에서 노출한 공개 형식에 접근할 수 있다.
- com.iteratrlearning.io 모듈에 의존하는 모든 모듈은 자동으로 com. iteratrlearning.core 모듈을 읽을 수 있게 된다.

```java
module com.iteratrlearning.ui {
	requires com.iteratrlearning.core;
	exports com.iteratrlearning.ui.panels; 
	exports com.iteratrlearning.ui.widgets to
		com.iteratrlearning.ui.widgetuser;
   }
```
- exports to를 이용하면 com. iteratrlearning.ui.widgets의 접근 권한을 가진 사용자의 권한을 com.iteratrlearning. ui.widgetuser로 제한할 수 있다.

```java
open module com.iteratrlearning.ui {
}
```
- 모듈 선언에 open 한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다.
- 모듈의 가시성에 다른 영향을 미치지 않는다.
- 리플렉션 때문에 전체 모듈을 개방하지 않고도 opens 구문을 모듈 선언에 이용해 필요한 개별 패키지만 개방할 수 있다. 
- exports-to로 노출한 패키지를 사용할 수 있는 모듈을 한정했던 것 처럼, open에 to를 붙여서 반사적인 접근을 특정 모듈에만 허용할 수 있다.

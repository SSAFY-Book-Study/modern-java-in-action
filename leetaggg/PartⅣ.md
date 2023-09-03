# 11장. null 대신 Optional 클래스

---

### NullPointerException

- 실제 값이 아닌 null을 가지고 있는 객체/변수를 호출할 때 발생하는 예외

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

public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}

// NullPointerException the return value of Person.getCar() is null
System.out.println(getCarInsuranceName(person));
```

### null 때문에 발생하는 문제

- **에러의 근원**
- **코드 가독성 문제**
- **의미가 없음**
- **자바 철학에 위배**
- **형식 시스템의 구멍을 만듦**

### 다른 언어의 null 처리

- 그루비 : 안전 내비게이션 연산자(?.) 호출 체인에 null인 참조가 있다면 null 반환

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

- 하스켈, 스칼라는 **선택형값**을 저장하는 Maybe, Option[T] 형식을 제공

## Optional 클래스

---

- 자바 8은 선택형값을 캡슐화하는 **Optional 클래스**를 제공
- null 참조 시 NPE가 발생하지만, 값이 없는 Optional은 객체이므로 다양한 방식으로 활용 가능
- 값이 있을 수도 있고, 없을 수도 있는 모델의 의미가 명확해짐

```groovy
// Optional 클래스 사용
public class Person {
	private Optional<Car> car;
	public Optional<Car> getCar() {	return car; }
}

public class Car {
	private Optional<Insurance> insurance;
	public Optional<Insurance> getInsurance() { return insurance; }
}

public class Insurance {
	private String name;
	public String getName() { return name; }
}
```

## Optional 객체 생성 - 정적 팩토리 메서드

---

**Optional.empty()**

- 빈 Optional 객체 생성

```java
Optional<Car> optCar = Optional.empty();
```

**Optional.of()**

- null이 아닌 값으로 Optional 객체 생성
- 값이 null이라면 NPE 발생

```java
Optional<Car> optCar = Optional.of(Car);
```

**Optional.ofNullable()**

- null 값을 저장할 수 있는 Optional 객체 생성

```java
Optional<Car> optCar = Optional.ofNullable(car);
```

## Optional 객체 조작

---

Optional 클래스도 map을 지원하지만, map을 통해 반환되는 객체는 Optional에 감싸져 반환됨

→ 반환되는 객체가 Optional일 경우, 반환 타입은 Optional<Optional>로 중첩됨

**flatMap()**

- 인수로 받은 함수를 적용해서 새로운 스트림으로 반환됨
- flatMap()을 통해 새로운 Optional 객체에 담아 반환함

**get()**

- Optional의 값을 읽는 메서드
- 래핑된 값이 있으면 해당 값을 반환하지만, 값이 없다면(null 이라면) **NoSuchElementException**을 반환함

※ NoSuchElementException : 데이터 구조나 컬렉션에서 요소를 찾을 수 없는 경우 발생

**orElse()**

- Optional에 값이 없을 시 인자로 받은 기본 값을 제공

**orElseGet(Supplier<? extends T> other)**

- Supplier 함수를 인자로 받아 Optional에 값이 없을 시 Supplier 함수 실행

**orElseThrow(Supplier<? extends X> exceptionSupplier)**

- Optional에 값이 없을 시 예외를 생성하여 발생

**ifPresent(Consumer<? super T> consumer)**

- Optional에 값이 존재하면 Consumer 함수 실행

**ifPresentOrElse()**

- ifPresent와 같지만, Optional이 비어있을 때 Runnable을 실행

## Optional 클래스의 사용

---

### 잠재적으로 null이 될 수 있는 값

- Optional.ofNullable()

```java
Object value = map.get("key"); // "key"에 해당하는 값이 없을 시 null 반환

// Optional로 null일 수도 있는 값을 안전하게 변환
Optional<Object> value = Optional.ofNullable(map.get("key"));

```

### 예외 처리 시

```java
public static Optional<Integer> stringToInt(String s) {
	try {
	return Optional.of(Integer.parseInt(s)); // 변환 가능 시 변환된 값을 Optional 객체에 담아 반환
	} catch (NumberFormatException e) {
	return Optional.empty(); // 빈 Optional 객체 반환
	}
}
```

### 기본형 Optional

- Optional도 기본형에 특화된 OptionalInt, OptionalLong 등이 존재
- 스트림과 달리 요소 수가 1개로 제환되므로 성능 개선 불가능
- 기본형 Optional의 경우 일반 Optional과 혼용이 불가능

→ 기본형 Optional 사용을 **지양**

## 도메인 모델에 Optional 사용

---

- Optional 클래스의 용도는 **선택형 반환값을 지원하는 것**
- Optional 클래스는  Serializable 인터페이스를 구현하지 않음
- 하지만 도메인 모델 구성 시 Optional 사용이 **바람직하며**, Optional로 값을 반환 받을 수 있는 메서드 추가 방식이 권장됨

# 12장. 새로운 날짜와 시간 API

---

## 자바 8 이전의 날짜와 시간 API 문제점

---

**java.util.Date**

- 날짜가 아닌 밀리초 단위 사용, 1900 기준의 오프셋, 0에서 시작하는 달 인덱스 등의 모호한 설계
- 반환되는 문자열의 추가 사용이 어려움
- JVM 기본시간대인 중앙 유럽 시간(CET) 사용

**java.util.Calendar**

- 남아있는 0에서 시작하는 달 인덱스 문제점
- DateFormat과 같은 일부 기능은 Date 클래스에서만 작동함
- 가변 클래스의 문제점으로 유지 보수가 어려움

→ **새로운 날짜와 시간 API의 필요성**

## LocalDate와 LocalTime

---

### LocalDate

- 날짜만을 표현하는 **불변 객체**
- 정적 팩토리 메서드 of로 인스턴스 생성
- 인스턴스로 연도, 달, 요일 등을 반환하는 메서드 제공
- get 메서드의 TemporalField를 전달하여 연, 월, 일 정보를 반환

**LocalDate.now()**

- 시스템 시계의 정보를 제공하는 정적 팩토리 메서드

---

### LocalTime

- 시간만을 표현하는 **불변 객체**
- 정적 팩토리 메서드 of로 인스턴스 생성

### LocalDateTime

- 날짜와 시간 **모두** 표현하는 불변 객체
- toLocalDate(), toLocalTime()으로 날짜나 시간의 인스턴스를 추출

## Instant

---

- 기계의 관점에서 시간을 표현하는 방법
- 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현
- 나노초의 정밀도를 제공하며, 사람이 읽을 수 있는 표현은 제공하지 않음

```java
// 정적 팩토리 메서드 ofEpochSecond
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후 1억 나노초
Instant.ofEpochSecond(4, -1_000_000_000); // 4초 이전 1억 나노초

int day = Instant.now().get(ChronoField.DAY_OF_MONTH); // UnsupportedTemporalTypeException 발생
```

## Duration과 Period

---

### Duration.between()

- 두 시간 객체 사이의 지속시간 생성

```java
// 두 시간 객체 사이 지속시간 Duration
Duration d1 = Duration.between(dateTime1, dateTime2); 
Duration d2 = Duration.between(instant1, instant2);
```

### Period.between()

- 두 날짜 객체 사이의 지속시간 생성

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11),
			LocalDate.of(2017, 9, 21));
```

## 날짜와 시간 객체 조정, 파싱

---

- 불변적인 날짜와 시간 객체가 아닌 일부 속성 변경
- withAttribute 메서드로 기존 LocalDate를 간단히 변경

### TemporalAdjusters

- with 메서드에 좀 더 다양한 동작을 수행하는 메서드

```java
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
```

### DateTimeFormatter, DateTimeFormatterBuilder

- 날짜, 시간을 파싱하는 클래스
- 미리 정의된 상수와 Builder을 통해 포매터를 제어하여 파싱할 수 있음

## 다양한 시간대와 캘린더

---

### ZoneRules

- 약 40개의 시간대를 제공하는 클래스
- 지역 ID로 특정 ZoneId를 구분
- UTC/GMT 기준으로 시간대 표현 가능

```java
// ZoneId는 "지역/도시" 형식으로 이루어짐
ZoneId romeZone = ZoneId.of("Europe/Rome");
```

# 13장. 디폴트 메서드

---

- 인터페이스의 새로운 메서드를 추가 시 해당 인터페이스를 구현했던 모든 클래스에 새로운 메서드를 구현해야하는 문제 발생
    - 인터페이스 내부의 **정적 메서드** 사용
    - 인터페이스의 기본 구현을 제공하는 **디폴트 메서드** 사용

```java
// 자바 8에서 추가된 List 인터페이스의 sort 디폴트 메서드
default void sort(Comparator<? super E> c){
	Collections.sort(this, c);
}
```

### 정적 메서드와 인터페이스

---

- 자바 8 이전에는 인터페이스의 인스턴스를 **유틸리티 클래스**로 정의한 **정적 메서드**를 통해 활용함
- 자바 8에는 인터페이스에 직접 정적 메서드를 선언할 수 있음
- 하지만 과거 버전 호환성을 위해 유틸리티 클래스가 남아있음

※ **정적 메서드 :** 인터페이스의 인스턴스를 활용하는 메서드

※ **유틸리티 클래스 :** 정적 메서드를 정의하는 클래스

### 프로그램 변경 시 호환성 문제

- **바이너리 호환성** : 변경 이후에도 에러 없이 기존 바이너리(인증, 준비, 해석)가 실행될 수 있는 상황
- **소스 호환성** : 코드를 고쳐도 기존 프로그램을 성공적으로 재 컴파일 할 수 있는 상황
- **동작 호환성** : 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행하는 상황

### 인터페이스에 새로운 메서드 추가 시 문제

- 바이너리 호환성은 유지되지만, 소스 호환성 문제가 발생

**→ 디폴트 메서드**를 통해 해결

## 디폴트 메서드란?

---

인터페이스 내에 있는 **구현 메서드**이며, 추상 메서드와 달리 **default 키워드**로 시작하고 **메서드 바디(구현부, {})**가 있어야함

### 추상 클래스 VS 자바 8의 인터페이스

- 클래스는 **하나의 추상 클래스**를 상속 가능, **인터페이스는 여러 개** 구현 가능
- 추상 클래스는 인스턴스 변수를 가질 수 있지만, 인터페이스는 가질 수 없음

## 디폴트 메서드의 활용

---

### 선택형 메서드

- 인터페이스를 구현하는 클래스에 기본 구현을 제공할 수 있으므로 **불필요한 코드를 줄임**

### 동작 다중 상속

- 기존 인터페이스를 수정하지 않고 디폴트 메서드를 활용하여 구현할 수 있음
- 필요한 기능을 선택하여 클래스를 쉽게 구현할 수 있음

## 해석 규칙

---

여러 인터페이스를 구현할 때 같은 시그니처의 디폴트 메서드가 포함된 인터페이스를 구현하는 상황에서의 규칙 3가지

1. 디폴트 메서드보다 클래스나 슈퍼 클래스에서 정의한 메서드가 우위
2. 1번 규칙 이외에는 서브인터페이스가 우위
3. 이외의 상황에서는 클래스에서 명시적으로 디폴트 메서드를 오버라이드하고 호출해야함

### 다이아몬드 문제

- 같은 디폴트 메서드를 상속받은 두 인터페이스를 구현할 때 2개의 디폴트 메서드가 충돌하는 문제
- 자바에서는 둘 중 하나의 디폴트 메서드를 명시적으로 호출하는 방식으로 해결

# 14장. 자바 모듈 시스템

---

### 모듈화의 필요성

- 이해하고 유지보수하기 쉬운 코드를 구현하는 것은 저수준의 영역
- 고수준의 영역(SW 아키텍쳐)에서는 추론하기 쉬워 생산성을 높이기 위해 **관심사 분리**와 **정보 은닉**이 필요

### 관심사 분리(SoC, Separation of Concerns)

- 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙
- 모듈이라는 각각의 부분을 겹치지 않는 코드 그룹으로 분리 가능
    - 개별 기능을 따로 작업할 수 있으므로 쉽게 협업이 가능
    - 개별 부분을 재사용하기 쉬움
    - 전체 시스템을 쉽게 유지보수할 수 있음

### 정보 은닉

- 세부 구현을 숨기도록 장려하는 원칙
- 코드를 **캡슐화**하여 결합도를 낮추고, 자바 9 이후로는 클래스와 패키지가 의도된 대로 공개되었는지 컴파일러가 확인 가능

## 자바 9 이전 모듈의 필요성

---

### 자바 9 이전 모듈화의 한계

**제한된 가시성 제어**

- 네 가지 가시성 접근자가 있지만, 패키지 간 가시성 제어 기능은 없는 수준

**클래스 경로**

- 클래스 경로에 같은 클래스를 구분하는 버전 개념이 없음
  - 같은 라이브러리의 다른 버전을 사용하는 상황이 발생
- 클래스 경로는 명시적인 의존성을 지원하지 않음
  - 자바 9 이전에는 직접 클래스 파일을 추가, 제거해보며 확인해야했음

### 거대한 JDK

- **자바 개발 키트(JDK)** :  자바 프로그램을 만들고 실행하는데 도움을 주는 도구의 집합
- JDK 라이브러리의 내부 API가 자바 언어의 낮은 캡슐화 지원 때문에 외부로 공개됨

  → 호환성을 깨지않고 관련 API를 바꾸기 어려움


## 자바 모듈 : 큰 그림

---

- 자바 8은 **모듈**이라는 새로운 자바 프로그램 구조 단위를 제공
- 모듈은 module 키워드에 이름과 바디를 추가해서 정의

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4cfdab8d-5161-4679-a04b-4fbe7c7e3db6/eb395520-b26d-4f7d-9a32-e125587fc386/Untitled.png)

- exports 부분을 돌출부(노출시키는 패키지), requires는 패인 부분(필요로 하는 모듈)

### 세부적인 모듈화와 거친 모듈화

- 세부적인 모듈화
  - 모든 패키지가 자신의 모듈을 갖음
  - 이득에 비해 설계 비용이 증가
- 거친 모듈화
  - 한 모듈이 시스템의 모든 패키지를 포함
  - 모듈화의 모든 장점을 잃음

→ 가장 좋은 방법은 시스템을 실용적으로 분해하며 이해하기 쉽고 고치기 쉬운 수준으로 적절하게 모듈화 되었는지 주기적으로 확인하는 프로세스를 갖는 것
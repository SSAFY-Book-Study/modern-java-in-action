## Chapter11 null 대신 Optional 클래스

자바로 프로그램을 개발하면서 한 번이라도 NullPointerException을 겪어 봤을 것이다
1965년 토니 호어라는 영국 컴퓨터과학자가 힙에 할당되는 레코드를 사용하며 형식을 갖는 최초의 프로그래밍 언어 중 하나인 알골을 설계하면서 처음 Null 참조가 등장했고, 그는 `구현하기가 쉬웠기 때문에 null을 도입했다`라고 그 당시를 회상한다고 한다
여러 해가 지난 후 호어는 당시 null 및 예외를 만든 결정을 가리켜 `십억 달러짜리 실수`라고 표현했다
`null` 때문에 어떤 문제가 발생할 수 있는지, 이를 방지하기 위해 Optional의 등장 및 활용을 알아보자

### 값이 없는 상황을 어떻게 처리할까?

자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩구조로 구현했다고 하자
```java
/* Person/Car/Insurance 데이터 모델 */

public class Person {
    private Car car;

    public Car getCar() {
        return car;
    }
}

public class Car {
    private Insurance insurance;

    public Insurance getInsurance() {
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
위의 구조를 기반으로
- 다음 코드에서는 어떤 문제가 발생할까?

```java
public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```
코드는 문제가 없어보이지만 `차를 소유하지 않은 사람`이 getCar를 호출하면? 
대부분의 프로그래머가 Car를 Null참조로 반환하는 방식으로 자동차 소유를 하지 않음을 표현할 것이다
그렇게 되면 Null 참조 객체에 `getInsurance()` 메서드를 호출하기 때문에 `NullPointerException`이 발생하면서 프로그램 실행이 중단된다

- 프로그램을 설계하면서부터 연쇄작용이 시작되었던 것이다..

### 보수적인 자세로 NPE 줄이기
예기치 않은 NullPointerException을 피하려면? 
대부분의 프로그래머는 필요한 곳에 다양한 Null 확인 코드를 추가해서 Null 예외 문제를 해결하려 할 것이다
-> 이를 보수적인 자세라 표현하고 있다

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
잦은 들여쓰기로 작업이 증가할수록 들여쓰기 레벨이 올라간다 이는 유지보수에 좋아보이지 않으며 다른 방법이 필요해보인다
- 모든 변수가 null인지 의심하므로 변수를 접근할 때 마다 중첩된 If -> `깊은 의심 deep doubt`

조금 다른 방법으로 중첩 if 블록을 없애보자
```java
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";    
    }

    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }

    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
```
null 변수가 존재하면 즉시 `Unknown`을 반환한다
하지만 이 예제도 그렇게 좋은 코드가 아니다
- 메서드에 네 개의 출구가 생김
- null일 때 반환되는 기본값 `Unknown`이 세 곳에서 반복되며, 문자열 오타 실수 여지가 있음

### Null 때문에 발생하는 문제
- 에러의 근원: NPE는 자바에서 가장 흔히 발생하는 에러
- 코드를 어지럽힘: 중첩된 Null 확인 코드를 추가해야 하므로 null 때문에 가독성이 떨어짐
- 아무 의미가 없다: null은 아무 의미도 표현하지 않음, **정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다**
- 자바 철학에 위배: 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는것이 Null 포인터
- 형식 시스템에 구멍을 만듦: null은 무형식이며 정보를 포함X, -> 모든 참조 형식에 Null을 할당할 수 있음 -> 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 Null이 어떤 의미로 사용되었는지 알 수 없다

### Optional 클래스 소개
- 자바 8은 하스켈과 스칼라의 영향을 받아 `java.util.Optional<T>`라는 새로운 클라스를 제공
- Optional은 선택형값을 캡슐화하는 클래스이며 값이 있든 없든 Optional로 감싸 NPE를 방지할 수 있음(Optional 객체를 반환하므로)

```java
/* Optional로 Person/Car/Insurance 데이터 모델 재정의 */
public class Person {
    private Optional<Car> car;

    public Optional<Car> getCar() {
        return car;
    }
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
- `Optional` 클래스를 사용하면서 모델의 의미 sementic가 더 명확해졌음을 확인할 수 있음
- 사람은 `Optional<Car>`, `Optional<Insurance>`를 참조하는데, 이는 사람이 자동차를 소유했을 수도, 자동차는 보험에 가입되어 있을 수도, 아닐 수도 있음을 명확히 설명

하지만 모든 Null 참조를 Optional로 대치하는 것은 바람직하지 않다
- Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것

### Optional 적용 패턴

#### Optional 객체 만들기
- 빈 Optional: `Optional<Car> optCar = Optional.empty();`
- null이 아닌 값으로 Optional 만들기: `Optional<Car> optCar = Optional.of(car);`
- null값으로 Optional만들기: `Optional<Car> optCar = Optional.ofNullable(car);` car가 Null이라면 빈 Optional 객체가 반환됨

### 맵으로 Optional의 값을 추출하고 변환하기
이전에 알아본 것처럼 정보에 접근하기 전에 null인지 확인해야하는 부분이 있었다
```java
String name = null;
if(insurance != null) {
  name = insurance.getName();
}
```
이런 유형의 패턴에 사용할 수 있도록 Optional은 map 메서드를 지원한다
```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```
- Optional의 map 메서드는 스트림의 map 메서드와 개념적으로 비슷
- Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있다

### flatMap으로 Optional 객체 연결
map을 사용하는 방법을 배웠으므로 다음처럼 Map을 이용해서 코드를 재구현할 수 있다
```java
Optional<Person> optionalPerson = Optional.of(person);
Optional<String> name = optionalPerson.map(Person::getCar)
                                      .map(Car::getInsurance)
                                      .map(Insurance::getName);
```
위 코드는 컴파일되지 않는다
- `Optional<person>`은 map 메서드를 호출할 수 있다
- getCar는 `Optional<Car>` 형식의 객체를 반환
  - map 연산의 결과가 `Optional<Optional<Car>>`
  - `getInsurance`메서드를 지원하지 못함

2차원 배열의 요소를 1차원 배열 요소로 반환할 때 `flatMap`으로 변환할 수 있었다
따라서 이중첩 Optional도 flatMap을 통해 일차원 Optional로 평준화해야 한다
```java
/* Optional로 자동차의 보험회사 이름 찾기 */
Optional<Person> optionalPerson = Optional.of(person);
String name = optionalPerson.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("UNKNOWN");  // <- 결과 Optional이 비어있다면 기본값으로 사용
```
1. Person을 우선 Optional로 감싼다
2. flatMap(Person::getCar)를 호출한다 -> getCar메서드는 Optional<Car>를 반환하므로 중첩 Optional이 생성되므로 이를 평준화하는 것이다 -> 평준화 과정은 두 Optional을 합치는 과정인데 둘 중하나라도 null이면 빈 Optional을 생성하는 연산
3. flatMap(Car::getInsurance)도 마찬가지이다
4. 그 결과 Optional<String>에서 map 연산을 통해 String을 받는데 orElse메서드를 통해 빈 Optional일 경우 기본값을 지정


### Optional 스트림 조작
자바9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optinal에 stream() 메서드를 추가했음

```java
/* 사람 목록을 이용해 가입한 보험 회사 이름 찾기 */
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
            .map(Person::getCar) // 사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
            .map(optCar -> optCar.flatMap(Car::getInsurance)) //flatMap 연산을 통해 Optional<Car>를 해당 Optional<Insurane>로 변환
            .map(optInsurance -> optInsurance.map(Insurance::getName)) // Optional<Insuracne>를 해당 이름의 Optional<String>으로 변환
            .flatMap(Optional::stream) // Stream<Optional<String>>을 이름을 포함하는 Stream<String>으로 변환
            .collect(Collectors.toSet());
}
```
스트림 요소 조작에서 변환, 필터 등의 일련의 여러 긴 체인이 필요한데 이 예제에서 Optional로 감싸있어 과정이 조금 더 복잡해짐
가장 큰 문제는
- Optional 덕분에 null 걱정 없이 연산을 처리할 수 있지만 마지막 `collect`연산에 비어있는 결과가 들어간다
- 빈 Optional을 제거하고 값을 언랩해야 한다는 문제가 존재

```java
Stream<Optional<String>> optionalStream = persons.stream()
                                                .map(Person::getCar)
                                                .map(optCar -> optCar.flatMap(Car::getInsurance))
                                                .map(optInsurance -> optInsurance.map(Insurance::getName));
Set<String> collect = optionalStream.filter(Optional::isPresent)
                                    .map(Optional::get)
                                    .collect(Collectors.toSet());
```
- Optional 클래스의 stream을 이용하면 한 번의 연산으로 같은 결과를 낼 수 있으며 각 Optional이 비어있는지 아닌지에 따라 Optional을 0개 이상의 항목을 포함하는 스트림으로 변환한다
- 따라서 두 수준의 스트림으로 변환하고 다시 한 수준의 평면 스트림으로 바꿔 collect하므로 성능이 더 떨어질 수 있다
- Optional.stream을 사용하면 같은 결과를 만들어낼 수 있다

```java
/* 두 방식 테스트 */
public class Main {
    public static void main(String[] args) throws IOException {
        List<Person> people = new ArrayList<>();
        people.add(new Person(new Car(new Insurance("insurance1"))));
        people.add(new Person(new Car(new Insurance("insurance2"))));
        people.add(new Person(new Car(new Insurance(null))));

        Set<String> set = getCarInsuranceNames(people);

        Set<String> collect = getCarInsuranceNames2(people);

        for (String name : set) {
            System.out.println("set: "+name);
        }

        for (String name : collect) {
            System.out.println("collect: "+name);
        }
    }
    public static Set<String> getCarInsuranceNames(List<Person> persons) {
        return persons.stream()
                .map(Person::getCar) // 사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
                .map(optCar -> optCar.flatMap(Car::getInsurance)) //flatMap 연산을 통해 Optional<Car>를 해당 Optional<Insurane>로 변환
                .map(optInsurance -> optInsurance.map(Insurance::getName)) // Optional<Insuracne>를 해당 이름의 Optional<String>으로 변환
                .flatMap(Optional::stream) // Stream<Optional<String>>을 이름을 포함하는 Stream<String>으로 변환
                .collect(Collectors.toSet());
    }
    
    public static Set<String> getCarInsuranceNames2(List<Person> persons) {
        Stream<Optional<String>> optionalStream = persons.stream()
                .map(Person::getCar)
                .map(optCar -> optCar.flatMap(Car::getInsurance))
                .map(optInsurance -> optInsurance.map(Insurance::getName));
        return optionalStream.filter(Optional::isPresent)
                .map(Optional::get)
                .collect(Collectors.toSet());
    }
}

class Person {
    private Optional<Car> car;

    public Person(Car car) {
        this.car = Optional.ofNullable(car);
    }

    public Optional<Car> getCar() {
        return car;
    }
}

class Car {
    private Optional<Insurance> insurance;

    public Car(Insurance insurance) {
        this.insurance = Optional.ofNullable(insurance);
    }
    public Optional<Insurance> getInsurance() {
        return insurance;
    }
}

class Insurance {
    private String name;

    public Insurance(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
/*
sout

set: insurance2
set: insurance1
collect: insurance2
collect: insurance1

*/
```

### 디폴트 액션과 Optional 언랩
빈 Optional에 대해서 기본값을 반환하도록 orElse 제공 외에도 Optional 인스턴스에 포함된 값을 읽는 다양한 방법 제공
- get()은 값을 읽는 가장 간단한 메서드이면서 동시에 가장 안전하지 않은 메서드
  - get은 래핑된 값이 있으면 반환 vs 없으면 `NoSuchElementException`
  - Optional에 반드시 값이 있는게 아니면 사용하지 않는 것이 바람직
- orElse를 이용하면 Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있음
- orElseGet은 orElse 메서드에 대응하는 게으른 버전의 메서드
  - Optional에 값이 없을때만 Supplier가 실행됨
  - 디폴트 메서드를 만드는 데 시간이 걸리거나(효율성), Optional이 비었을 때만 기본값을 생성하고 싶다면 사용
 
- orElseThrow를 이용하면 Optional이 비어있을 때 예외를 발생시킨다
- ifPresent를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다, 값이 없으면 아무일도 일어나지 않음
- ifPresentOrElse 이 메서드는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받음

### 두 Optional 합치기
- 두 Optional을 받아서 하나의 Optional을 반환해야하는 경우가 있다

```java
/* 두 Optional을 인수로 받아서 Optional<Insurance>를 반환하는 null 안전 버전 */

public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
        return Optional.of(new Insurance());
    }
    else return Optional.empty();
}
```
위 코드의 장점은 person과 car의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 정보를 명시적으로 보여준다

```java
/* map과 flatMap 메서드를 이용한 버전 */

public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
  return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```
- flatMap을 사용하면 person이 비어있지 않을 때만 findCheapestInsurance의 인자로 p를 사용
- car가 값을 포함하지 않는다면 빈 Optional을 반환하고 값이 존재한다면 map 메서드로 안전하게 전달한다

### 필터로 특정값 거르기
- Insurance 객체가 null인지 여부를 확인한 다음 getName 메서드를 호출해야 한다

Optional 객체에 filter 메서드를 이용해서 구현할 수 있다
```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "XXX".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```
- Optional이 비어있다면 Filter 연산은 아무 동작도 하지 않음
- 프레디케이트 결과가 False면 값은 사라져버리고 Optional은 빈 상태가 된다

![스크린샷 2023-09-02 오후 2 31 04](https://github.com/dpwns523/modern-java-in-action/assets/84260096/820dadd4-d20d-48fb-8dbd-5815fdbcfc7c)

### Optional을 사용한 실용 예제

#### 잠재적으로 Null이 될 수 있는 대상을 Optional로 감싸기
- 기존 자바API는 null을 반환하면서 요청한 값이 없거나 어떤 문제로 계산에 실패했음을 알림
- null을 반환하는 것보다 Optional을 반환하는 것이 더 바람직
  - map.get("key") 는 키에 해당하는 값이 map에 없으면 null을 반환
  - Optional로 감싸서 개선 가능
- if-then-else를 추가하거나 Optional.ofNullable 이용

#### 예외와 Optional zmffotm
- 자바 API 에서는 어떤 이유에서 값을 제공할 수 없을때 예외를 발생했다
- Integer.parseInt와 같은 예를 보면 문자열을 정수로 반환할수 없는 경우 NumberFormatException을 반환 한다

이러한 경우를 Optional을 활용해 개선해보자

```java
public static Optional<Integer> stringToInt(String s) {
  try {
    return Optional.of(Integer.parseInt(s));    // <- 문자열을 정수로 변환할 수 있으면 정수로 변환된 값을 포함하는 Optional 반환
  } catch (NumberForamtException e) {
    return Optional.empty();  // <- 그렇지 않으면 빈 Optional반환
  }
}
```

#### 기본형 Optional을 사용하지 말아야 하는 이유
스트림처럼 Optional도 기본형 특화 OptionalInt, OptionalLong 등의 클래스를 제공
- Optional의 최대 요소 수는 한 개이므로 기본형 특화 클래스로 성능을 개선할 수 없다
- Optional 클래스의 유용한 메서드 map, flatMap, filter등을 지원하지 않아 권장하지 않음
- 일반 Optional과 호환되지 않음

### 정리
- 역사적으로 프로그래밍 언어에서 Null 참조로 값이 없는 상황을 표현해옴
- 자바 8에서는 값이 있거나 없음을 표현할 수 있는 `java.util.Optional<T>`를 제공
- 팩토리 메서드 `Optional.empty`, `Optional.of`, `Optional.ofNullable` 등을 이용해 객체 생성 가능
- Optional 클래스는 스트림과 비슷한 연산을 수행하는 map, flatMap, filter등의 메소드를 제공한다
- Optional로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다 즉, Optional로 예상치 못한 NPE를 방지할 수 있다
- Optional을 활용하면 더 좋은 API를 설계할 수 있음 즉, 사용자는 메서드의 시그니처만 보고 Optional값이 사용되거나 반환되는지 예측할 수 있다

## Chapter12 새로운 날짜와 시간 API

자바8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공
`java.util.Date`는 자바 1.0에서부터 제공한 클래스로 
- 특정 시점을 날짜가 아닌 밀리초로 표현
- 1900년을 기준으로 하는 오프셋
- 0에서 시작하는 달 인덱스 등 모호한 설계
- 결과가 밀리초로 직관적이지 못함
`Date` 클래스에 문제가 있음을 알고 있었지만 하위 버전에 대한 호환성을 깨뜨리지 않고 해결할 수 없었기에 새로운 클래스를 제공하기 시작하였음

### LocalDate, LocalTime, Instant, Duration, Period 클래스

LocalDate, LocalTime, Instant, Duration, Period 클래스를 사용하여 간단한 날짜와 시간 간격을 정의할 수 있다

### LocalDate와 LocalTime 사용

- LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변객체이다
- LocalDate 객체는 어떤 시간대 정보도 포함하지 않는다
- 정적 팩토리 메소드 of를 활용하여 LocalDate 인스턴스를 만들 수 있다
- 팩토리 메소드 now()는 시스템 시계의 정보를 이용해 현재 날짜 정보를 얻는다

```java
LocalDate date = LocalDate.of(2017, 9, 21)
date.getYear()
...
```
- 시간대를 포함한 클래스는 LocalTime을 사용한다.
```java
LocalTime time = LocalTime.of(13, 45, 20) // 13:45:20
time.getHour()
...
```
- 날짜와 시간 문자열로 LocalDate와 LocalTime의 인스턴스를 만드는 방법도 있다.
```java
LocalDate date = LocalDate.parse("2017-09-21")
LocalTime time = LocalTime.parse("13:45:20")
```
- parse 메서드에 DateTimeFormatter를 전달할 수 있다.

### 날짜와 시간 조합
LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다
따라서 날짜와 시간을 모두 표현할 수 있다
```java
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);

dt1.toLocalDate();    // <- 2017-09-21
dt2.toLocalTime();    // <- 13:45:20;
```

### Instant 클래스 : 기계의 날짜와 시간
Instant 클래스는 유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 시점까지의 시간을 초로 표현
기계의 관점에서는 연속된 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 표현 방법
- 팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다
- Instant 클래스는 나노초의 정밀도를 제공한다

### Duration과 Period 정의

- Temporal 인터스페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다
- Duration클래스는 between으로 두시간 객체 사이의 지속시간을 만들수 있다
```java
Duration d1 = Duration.between(time1, time2);
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d1 = Duration.between(instant1, instant2);
```
- LocalDateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어졌다
- Period 클래스의 팩토리 메서드 between을 이용하면 두 LocalDate의 차이를 확인할 수 있다
```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));
```
- Duration과 Period 클래스는 자신의 인스턴스를 만들 수 있도록 다양한 팩토리 메서드를 제공한다.
```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
```
- 지금까지 살펴본 모든 클래스는 불변이다. 불변 클래스는 함수형 프로그래밍과 스레드 세이프하여 도메인 모델의 일관성을 유지하는데 좋은 특성이다
- 새로운 날짜와 시간 API에서는 변경된 객체 버전을 만들 수 있는 메서드를 제공한다

### 날짜 조정, 파싱, 포매팅

- withAttribte 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단히 만들 수 있다
- 모든 메서드는 기존 객체를 변경하지 않는다
```java
/* 절대적인 방식으로 LocalDate의 속성 바꾸기 */

LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.withYear(2011); // 2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); // 2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); // 2011-02-25
```

- 마지막 행에서 보여주는 것처럼 첫 번째 인수로 TemporalField를 갖는 메서드를 사용하면 조금더 범용적으로 날짜를 변경할 수 있다
- 선언형으로 LocalDate를 사용하는 방법도 존재한다
```java LocalDate date1 = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = date1.plusWeek(1); // 2017-09-28
LocalDate date3 = date2.minusYear(6); // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); // 2012-03-28
```
### TemporalAdjusters 사용하기

- Java는 다음주 일요일, 돌아오는 평일등 복잡한 날짜 조정기능을 지원한다
- 날짜와 시간 API는 다양한 상황에서 사용할 수 있도록 TemporalAdjuster를 제공한다
```java
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));
LocalDate date3 = date2.with(lastDayOfMonth());
```

- 필요한 기능이 정의되어 있지 않은 때는 커스텀 TemporalAdjuster 를 구현할 수 있다
```java
@FunctionalInterface
public interface TemporalAdjuster {
  Temporal adjustInto(Temporal temporal);
}
```

- 함수형 인터페이스로 되어 있으므로 adjustInto만 구현하면 된다
```java
// Working Day 만 구하는 커스텀 TemporalAdjuster
@Override
public Temporal adjustInto(Temporal temporal) {
    DayOfWeek today = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK)); // 현재 날짜 읽기
    int dayToAdd = 1;
    if (today == DayOfWeek.FRIDAY) {    // <- 금요일이면 3일 추가
        dayToAdd = 3;
    } else if (today == DayOfWeek.SATURDAY) {    // <- 토요일이면 2일 추가
        dayToAdd = 2;
    }
    return temporal.plus(dayToAdd, ChronoUnit.DAYS);    // 적정한 날 수 만큼 추가된 날짜를 반환
}
```

### 날짜와 시간 객체 출력과 파싱
- 날짜와 시간 관련해서 포매팅과 파싱은 필수이다
- DateTimeFormmatter 클래스는 BASIC_ISO_DATE와 ISO_LOCAL_DATE등의 상수를 미리 정의하고 있다
- DateTimeFormmatter를 활용하여 날짜나 시간을 특정 형식의 문자열로 만들 수 있다
```java
LocalDate date = LocalDate.of(2014, 3, 18);
date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-18
```

- 반대로 날짜나 시간을 표현하는 문자열을 파싱하여 날짜 객체를 다시 만들 수 있다

```java
LocalDate parse = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate parse2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```

- DateTimeFormmatter는 `Thread safe`한 클래스이며 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다.
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate localDate = LocalDate.of(2014, 3, 18);
String formattedDate = localDate.format(formatter);
LocalDate parse1 = LocalDate.parse(formattedDate, formatter);
```

- 지역화 된 DateTimeFormatter도 만들수 있으며 조금더 복합적인 포매터를 만들기 위해 Builter를 사용할 수 있다
```java
// d. MMMM yyyy

DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
                .appendText(ChronoField.DAY_OF_MONTH)
                .appendLiteral(". ")
                .appendText(ChronoField.MONTH_OF_YEAR)
                .appendLiteral(" ")
                .appendText(ChronoField.YEAR)
                .parseCaseInsensitive() // 정해진 형식과 정확하게 일치하지 않아도 해석가능
                .toFormatter(Locale.ITALIAN);
```

### 정리
- Java 8 이전에서 제공하는 기존 Date 클래스와 관련 클래스는 결함이 존재했다
- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변클래스이다
- 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있으며 기존 인스턴스를 변환하지 않도록 처리 결과로 새로운 인스턴스가 생성된다
- TemporalAdjuster를 이용하면 단순히 값을 바꾸는 것 이상의 복잡한 동작을 수행할수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의 할 수 있다. 패턴을 이용할 수 있고 포매터는 스레드 세이프하다
- 특정 지역/장소에 상대적인 시간대 또는 UTC/GMT 기준 오프셋을 이용해 시간대를 정의할 수 있다
- ISO-8601 표준 시스템을 준수하지 않는 캘린더 시스템도 이용할 수 있다

## Chapter13 디폴트 메서드

전통적인 자바에서 인터페이스와 관련 메서드는 한 몸처럼 구성된다
인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 슈퍼클래스의 구현을 상속받아야 한다

자바8 API에도 List 인터페이스에 sort 같은 메서드를 추가했다면 모든 클래스의 구현도 고쳐야 하는 문제가 발생한다

- 자바 8에서는 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법을 제공
- 인터페이스 내부 **정적 메서드**를 사용
- 기본 구현을 제공할 수 있도록 **디폴트 메서드** 기능을 사용하는 것

### 변화하는 API
API를 바꾸는 것이 왜 어려운지 예제를 통해 살펴본다

### API 버전 1
- 모양의 크기를 조절하는데 필요한 `setWidth, setHeight, getWidth, setAbsoluteSize` 등의 메서드를 정의하는 Resizable 인터페이스
- Resizable 인터페이스를 구현하는 Ellipse라는 클래스

```java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}
```

### API 버전2
- Resizable을 구현하는 `Square`와 `Rectangle` 구현을 개선해달라는 요청

```java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor);    // <- 추가된 메서드
}
```

### 사용자가 겪는 문제

- Resizable을 고치면 Resizable을 구현하는 모든 클래스는 추가된 메소드 또는 변경된 메소드를 구현해야 한다
- 인터페이스에 새로운 메서드를 추가하면 `바이너리 호환성`은 유지되지만 `컴파일 단계에서 에러`가 발생할 것이다
- 공개된 API를 고치면 `기존 버전과의 호환성 문제가 발생`했다
디폴트 메서드가 나온 이후로는 이러한 문제들을 해결할 수 있게 되었다

> 바이너리 호환성: 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일어나지 않는 것
 
> 소스 호환성: 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음 -> 인터페이스에 메서드를 추가하면 소스 호환성이 아님 -> 추가한 메서드를 구현하도록 클래스 수정이 일어남

> 동작 호환성: 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 실행 -> 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지

### 디폴트 메서드란 무엇인가?
- 자바 8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 디폴트 메서드를 제공

```java
public interface Sized { 
  int size();
  default boolean isEmpty() { // <- 디폴트 메서드
    return size() == 0;
  }
}
```
- 디폴트 메서드는 추상 메서드에 해당하지 않으므로 **함수형 인터페이스**에는 여러 디폴트 메서드를 추가할 수 있다

### 디폴트 메서드 활용 패턴

디폴트 메서드를 이용하는 두가지 방식
- 선택형 메서드 Optional Method
- 동작 다중 상속 Multiple Inheritance of Behavior

#### 선택형 메서드
- 인터페이스를 구현하는 클래스에서 메서드의 내용이 비어있는 상황, 인터페이스에 정의된 메서드이지만 실제로 사용하지 않는 메서드라 빈 메서드를 정의해야 하는 상황
- 디폴트 메서드를 이용하면 이러한 메서드에 기본 구현을 제공해줄 수 있다
Iterator 인터페이스를 살펴보자
```java
interface Iterator<T> {
  boolean hasNext();
  T next();
  default void remove() {
    throw new UnsupportedOperationException();
  }
}
```
기본 구현이 제공되므로 빈 remove 메서드를 구현할 필요가 없어졌고, 불필요한 코드를 줄일 수 있다

#### 동작 다중 상속

디폴트 메서드를 이용하면 기존에는 불가능했던 동작 다중 상속 기능도 구현할 수 있다
- 자바에서 클래스는 한 개의 클래스만 상속할 수 있지만 인터페이스는 여러 개 구현할 수 있다

```java
public class ArrayList<E> extends AbstractList<E>    // <- 한 개의 클래스를 상속
 implements List<E>, RandomAeccess, Cloneable, Serialiizable {}    // <- 네 개의 인터페이스를 구현
```

- 다중 상속 형식
    - ArrayList는 AbstractList, List, RandomAeccess, Cloneable, Serialiizable의 서브 형식이 된다
    - Java8 에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작을 상속받을 수 있다, 중복되지 않는 최소한의 인터페이스를 유지한다면 재사용과 조합을 쉽게 할 수 있다
    - 
#### 기능이 중복되지 않는 최소의 인터페이스

- 기존의 코드를 재사용하며 새로운 기능을 제공할때 디폴트 메서드를 활용할 수 있음
- 인터페이스를 조합해서 필요한 다양한 클래스를 구현할 수 있음

### 해석 규칙
만약 어떤 클래스가 같은 디폴트 메서드 시그니처를 포함하는 두 인터페이스를 구현하는 상황이라면?

같은 시그니처를 갖는 디폴트 메서드를 상속받는 상황이 생길 수 있으며 이런 상황에서는 어떤 인터페이스의 디폴트 메서드를 사용하게 되는지, 자바 컴파일러가 이러한 충돌을 어떻게 해결하는지 설명한다

#### 알아야 할 세 가지 해결 규칙

1. 클래스가 항상 이긴다. 클래스나 슈퍼 클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다
    - 클래스에서 오버라이드하는 것이 무조건 이긴다 
2. 1번 규칙 이외의 상황에서는 서브 인터페이스가 이긴다
    -  상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브 인터페이스가 이긴다
        -  즉 B가 A를 상속받는다면 B가 A를 이긴다
3. 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다
    - 디폴트 메서드가 같은 두 인터페이스 사이에 어떤 상속관계도 없다면 자바 컴파일러는 에러를 발생
    - 명시적으로 어떤 인터페이스의 디폴트 메서드를 호출할 것인지 선택해야 함
  
### 정리
- Java 8의 인터페이스는 구현 코드를 포함하는 디폴트 메서드, 정적 메서드를 정의할 수 있다
- 디폴트 메서드의 정의는 defualt 키워드로 시작하며 일반 메서드처럼 바디를 갖는다
- 공개된 인터페이스에 추상 메서드를 추가하면 소스 호환성이 깨진다
- 디폴트 메서드를 사용하면 소스 호환성을 유지할 수 있다
- 선택형 메서드와 동작 다중 상속에도 디폴트 메서드를 사용할 수 있다
- 클래스가 같은 시그니처를 갖는 여러 디폴트 메서드를 상속하며 생기는 충돌 문제를 해결하는 규칙이 있다
- 충돌 문제를 해결하는 규칙 3가지

## Chapter14 자바 모듈 시스템

자바 9에서 가장 많이 거론되는 새로운 기능이 모듈 시스템이다
자바 모듈 시스템은 한 권의 책으로 써야할 만큼 복잡한 주제
모듈을 사용하는 주요 동기를 이해하고 자바 모듈을 어떻게 사용할 수 있는지 빠르게 살펴볼 수 있도록 넓게 내용을 다뤄본다

### 압력: 소프트웨어 유추
모듈 시스템은 어떤 문제를 해결할 수 있는가?
지금까지 알아본 내용은 이해하고 유지보수하기 쉬운 코드를 구현하는 데 사용할 수 있는 새로운 언어 기능을 소개
하지만 이는 저수준의 영역에 해당하는 언어 기능

- 궁극적으로 소프트웨어 아키텍처 즉 고수준에서는 기반 코드를 바꿔야 할 때 유추하기 쉬우므로 생산성을 높일 수 있는 소프트웨어 프로젝트가 필요
- 추론하기 쉬운 소프트웨어를 만드는 데 도움을 주는 **관심사 분리**와 **정보 은닉**

### 관심사분리(SoC, Speratation of Concerns)

컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙
- 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다
- 개별 부분을 재사용하기 쉽다
- 전체 시스템을 쉽게 유지보수할 수 있다

### 정보 은닉

세부 구현을 숨김으로 코드를 관리하고 보호하는 데 유용한 원칙이다
캡슐화된 코드는 다른 부분과 고립되어 있어 내부적인 변화가 의도치않게 영향을 미칠 가능성이 줄어든다

자바에서는 클래스 내의 컴포넌트에 적절하게 Private 키워드를 사용했는지를 기준으로 컴파일러를 이용해 캡슐화를 확인할 수 있다

### 자바 모듈 시스템을 설계한 이유

#### 모듈화의 한계
자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는데 한계가 있었다. 자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공하는데, 이 중 패키지와 JAR 수준에서는 캡슐화가 거의 지원되지 않았다

#### 제한된 가시성 제어

한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 선언해야 한다. 결과적으로 이들 클래스와 인터페이스가 모두 공개되고, 사용자가 이 내부 구현을 마음대로 사용할 수 있게 된다

#### 클래스 경로

자바에서는 클래스를 모두 컴파일 한 다음 JAR 파일에 넣고 클래스 경로(class path)에 이 JAR 파일을 추가하여 번들로 사용할 수 있다

이러한 클래스 경로와 JAR 조합에는 몇 가지 약점이 존재한다

- 클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다
    - 예를들어 파싱 라이브러리의 JSONParser 클래스를 지정할 때 버전 1.0인지 2.0인지 지정할 수 없으므로 클래스 경로에 두 버전의 같은 라이브러리가 존재하면 어떤 문제가 발생할 지 예측할 수 없다
- 클래스 경로는 명시적인 의존성을 지원하지 않는다
    - 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용한다고 명시적으로 의존하지 않기 때문에 누락된 JAR를 확인하기 어렵고, 충돌이 발생할 수 있다
    - JVM이 `ClassNotFoundException`에러를 발생시키지 않고 정상적으로 실행할 때까지 경로 수정이 필요

### 거대한 JDK
- 자바 개발 키트(JDK)는 자바 프로그램을 만들고 실행하는데 도움을 주는 도구의 집합
- 하지만 JDK 의 내부 API 는 공개되지 않아야 하는데 낮은 캡슐화 지원으로 외부에 공개 `ex: sun.misc.Unsafe`
- 이러한 문제로 호환성을 깨지 않고는 관련 API를 바꾸기 어려운 상황이 되었음

### OSGI와 비교

- 자바 9에서 모듈화 기능이 추가되기 전에도 자바에는 OSGi(Open Service Gateway initiative) 는 모듈 시스템이 존재
- 자바 9 모듈 시스템과 OSGi 는 상호 배타적인 관계는 아니므로 공존이 가능

번들이라 불리는 OSGi 모듈은 특정 OSGi 프레임워크 내에서만 실행됨 (ex. 아파치 펠릭스, 에퀴녹스)
OSGi 프레임워크 내에서 애플리케이션을 실행할 때 원격으로 개별 번들을 설치, 시작, 중지, 갱신, 제거가 가능
시스템을 재시작하지 않아도 하위 부분을 핫스왑할 수 있다는 점이 강점이며, 같은 번들의 다른 버전 설치도 가능

### 자바 모듈 : 큰 그림
자바 8에서는 모듈이라는 새로운 자바 프로그램 구조 단위를 제공

모듈은 module 이라는 새 키워드에 이름과 바디를 추가해서 정의

모듈 디스크립터는 module-info.java라는 파일에 저장되고, 보통 패키지와 같은 폴더에 위치
![스크린샷 2023-09-03 오후 6 59 19](https://github.com/dpwns523/modern-java-in-action/assets/84260096/215b19c1-95f6-4bd0-96ee-e0062b39ae8e)


### 여러 모듈 활용하기

#### expotrs 구문
exports는 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다

기본적으로 모듈 내의 모든 것은 캡슐화되며, 다른 모듈에서 사용할 수 있는 기능만 무엇인지 명시적으로 결정해야함

```java
module expense.readers {
  //exports 패키지명
  exports com.example.expense.readers;
  exports com.example.expense.readers.file;
  exports com.example.expense.readers.http;
}
```

#### requires 구문
requires는 의존하고 있는 모듈을 지정한다

```java
module expense.readers {
  //requires 모듈명
  requires java.base;
}
```

#### 이름 정하기
오라클에서는 패키지명처럼 인터넷 도메인명을 역순으로 모듈의 이름을 정하도록 권고하고있다

모듈명은 노출된 주요 API 패키지와 이름이 같아야하며, 모듈이 패키지를 포함하지 않거나 어떤 다른 이유로 노출된 패키지 중 하나와 이름이 일치하지 않는 상황을 제외하면 모듈명은 작성자의 인터넷 도메인 명을 **역순으로 시작**해야 한다

### 컴파일과 패키징
메이븐을 이용해 컴파일한다면, 먼저 각 모듈에 pom.xml을 추가해야 한다. 또한 전체 프로젝트 빌드를 조정할 수 있도록 모든 모듈의 부모 모듈에도 pom.xml을 추가하고 의존성을 기술해야 한다

### 자동 모듈
`httpClient` 외부 라이브러리도 의존성을 기술하여 모듈로 사용할 수 있다. 모듈화가 되어있지 않은 라이브러리도 자동 모듈이라는 형태로 적절하게 변환한다. 다만, 자동 모듈은 암묵적으로 자신의 모든 패키지를 노출한다

### 모듈 정의와 구문들

#### requires transitive
다른 모듈이 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정할 수 있다
```java
module com.iteratrlearning.ui {
  requires transitive com.iteratrlearning.core;
  
  export com.iteratrlearning.ui.panels;
  export com.iteratrlearning.ui.widgets;
}

module com.iteratrlearning.application {
  requires com.iteratrlearning.ui;
}
```
결과적으로 com.iteratrlearning.application 모듈은 com.iteratrlearning.core에서 노출한 공개형식에 접근할 수 있다

#### exports to
exports to 구문을 이용해 가시성을 좀 더 정교하게 제어할 수 있다
```java
module com.iteratrlearning.ui {
  requires com.iteratrlearning.core;
  
  export com.iteratrlearning.ui.panels;
  export com.iteratrlearning.ui.widgets to
    com.iteratrlearning.ui.widgetuser;
}
```
- com.iteratrlearning.ui.widgets의 접근 권한을 가진 사용자의 권한을 com.iteratrlearning.ui.widgetuser로 제한할 수 있다.

#### open과 opens
모듈 선언에 open 한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다

`리플렉션` 때문에 전체 모듈을 개방하지 않고도 opens 구문을 모듈 선언에 이용해 필요한 개별 패키지만 개방할 수 있다
```java
open module com.iteratrlearning.ui {
}
```

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



# 3장

## 3장 스트림과 람다를 이용한 효과적 프로그래밍

### 1. 컬렉션 API 개선

### 1.1 컬렉션 팩토리

- 한계 :
    - 매끄럽지 X.
    - 내부적으로 불필요한 객체 할당을 필요로 함.
    - 결과는 변환할 수 있는 집합.
- 해결방안 : java9에서는 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 팩토리 메서드를 제공.

---

### 1.2 리스트 팩토리

- `List.of(요소1, 요소2, …)` 팩토리 메소드 활용.
- 변경할 수 없는 리스트 생성 ⇒ 컬렉션이 의도치 않게 변하는 것 방지.
    - 갱신, 추가, 삭제 모두 UnsupportedOperationException이 발생.
    - 리스트를 변경해야하는 상황이라면 다시 리스트 생성해야함.
- null 요소 금지 ⇒ 의도치 않은 버그 방지, 보다 간결한 내부구조.
- 데이터 처리 형식을 설정하거나 데이터를 변환할 필요 없다면, 팩토리 메서드 활용.

```java
List<String> cars = List.of("아반테", "소나타", "그랜저");
cars.set(0, "K3");// UnsupportedOperationException 발생!
cars.add("K5");// UnsupportedOperationException 발생!
```

---

### 1.3 집합 팩토리

- `Set.of(요소1, 요소2, …)` 팩토리 메소드 활용.
- 변경할 수 없는 리스트 생성 ⇒ 컬렉션이 의도치 않게 변하는 것 방지.
    - 갱신, 추가, 삭제 모두 UnsupportedOperationException이 발생.
    - 리스트를 변경해야하는 상황이라면 다시 리스트 생성해야함.
- 고유한 요소만 포함할 수 있음
    - 중복요소가 포함된 집합 생성시 IllegalArgumentException이 발생.

```java
Set<String> cars = Set.of("아반테", "소나타", "그랜저");
cars.add("K3");// UnsupportedOperationException
cars.remove("아반테");// UnsupportedOperationException
Set<String> overlapCars = Set.of("아반테", "아반테", "소나타", "그랜저");// IllegalArgumentException
```

---

### 1.4 맵 팩토리

- Map.of(키1, 값1, 키2, 값2, …) 팩토리 메소드 활용.
- 변경할 수 없는 리스트 생성 ⇒ 컬렉션이 의도치 않게 변하는 것 방지.
    - 갱신, 추가, 삭제 모두 UnsupportedOperationException이 발생.
    - 리스트를 변경해야하는 상황이라면 다시 리스트 생성해야함.
- 요소의 개수가 적은 경우(10개 이하)에만 활용 권장.

```java
Map<String, String> carInfo = Map.of("name", "아반테", "price", "23,000,000", "options", "2.0 Engine, SunRoof");
carInfo.put("fuel", "gasoline");// UnsupportedOperationException
carInfo.remove("options");// UnsupportedOperationException
```

- `Map.ofEnties(entry(키1, 값1), entry(키2, 값2), …)` 팩토리 메소드 활용.
- 요소의 개수가 적지 않은 경우(10개 초과)에 활용 권장.

```java
Map<String, String> carInfo = Map.ofEntries(
    Map.entry("name", "아반테"),
    Map.entry("price", "23,000,000"),
    Map.entry("options", "2.0 Engine, SunRoof")
);
```

---

### 1.5 리스트와 집합 처리

- 문제 : 컬렉션을 바꾸는 동작은 에러를 유발하며 복잡함.
- 해결방안 : java8의 List, Set 인터페이스에 추가된 메소드로 해결.
    - `removeIf` : Predicate를 만족하는 요소 제거( List, Set 구현 및 구현을 상속받은 곳에서 사용가능 ).
    - `replaceAll` : 리스트에서 이용할 수 있는 기능으로, UnaryOperator 함수를 이용해 요소를 변경.
    - `sort` : List 인터페이스에서 제공하는 기능으로 리스트 정렬.

- removeIf
    - 문제 : for-each문으로 요소를 반복하며 요소를 제거하는 경우 ⇒ ConcurrentModificationException 발생.
    - 원인 : 반복자(Iterator)의 상태는 컬렉션(cars)의 상태와 서로 동기화되지 않기 때문.
    - 해결 : Iterator 객체를 명시적으로 사용하고 그 객체의 remove()를 호출.
    - 활용 : removeIf() 메소드를 통해 간소화.

```java
List<String> cars = new ArrayList<>(){{
    add("아반테"); add("소나타"); add("그랜저");
}};

for (String car : cars) {
    if (car.equals("아반테")) {
        cars.remove(car);// ConcurrentModificationException 발생!
    }
}

/* 위의 코드 해석 */
for ( Interator<String> iterator = cars.iterator(); iterator.hasNext(); ) {
    String car = iterator.next();
    if (car.equals("아반테")) {
        cars.remove(car);
    }
}
/* 해결 */
for ( Iterator<String> iterator = cars.iterator(); iterator.hasNext(); ) {
    String car = iterator.next();
    if (car.equals("아반테")) {
        iterator.remove();
    }
}
/* removeIf */
cars.removeIf(car -> car.equals("아반테"));
```

- replaceAll
    - 문제 : 스트림 API에 map 메소드를 이용해 변경한다면 ⇒ 새 컬렉션을 만들어 버림.
    - 해결 : iterator의 set() 메소드를 활용.
    - 활용 : replaceAll() 메소드를 통해 간소화.

```java
/* 기존의 객체에 요소만 변경 */
for (ListIterator<String> iterator = cars.listIterator(); iterator.hasNext(); ){
    String car = iterator.next();
    iterator.set(car + " 풀옵션");
}

/* replaceAll */
List<String> cars = new ArrayList<>(){{
    add("아반테");
    add("소나타");
    add("그랜저");
}};

cars.replaceAll(car -> car + " 풀옵션");
```

---

### 1.6 맵 처리

- java8에서 추가된 Map 인터페이스의 디폴트 메서드.
- `forEach()` :
    - Map.Entry<K, V> 를 활용해 forEach 맵에서 키와 값을 반복하면서 확인하는 작업이 필요한 경우 해결.

```java
Map<String, String> carInfo = Map.ofEntries(
    Map.entry("name", "아반테"),
    Map.entry("price", "23,000,000"),
    Map.entry("options", "2.0 Engine, SunRoof")
);

/* Map.Entry 활용 */
for (Map.Entry<String, String> entry: carInfo.entrySet()) {
    String key = entry.getKey();
    String value = entry.getValue();
    System.out.println(key + ":" + value);
}
/* forEach */
carInfo.forEach((key, value) -> System.out.println(key + ":" + value));
```

- 정렬 메소드
    - Entry.comparingByValue : 맵의 항목을 값 기준으로 정렬.
    - Entry.comparingByKey : 맵의 항목을 키 기준으로 정렬.

```java
carInfo.entrySet().stream()
    .sorted(Map.Entry.comparingByKey())
    .forEachOrdered(System.out::println);
```

[](https://devbksheen.tistory.com/entry/%EB%AA%A8%EB%8D%98-%EC%9E%90%EB%B0%94-%EC%BB%AC%EB%A0%89%EC%85%98-%ED%8C%A9%ED%86%A0%EB%A6%ACCollection-Factory-%EC%86%8C%EA%B0%9C-%EB%B0%8F-%EC%82%AC%EC%9A%A9%EB%B2%95#getOrDefault%C-%A-)

- getOrDefault
    - 기존에 찾으려는 키가 존재하지 않는 경우, 기본값을 반환하여 NullPointerException 방지.
    - 키가 존재하더라도, 값이 null이라면 getOrDefault라도 null 반환이 가능함.

```java
Map<String, String> carInfo = Map.ofEntries(
    Map.entry("name", "아반테"),
    Map.entry("price", "23,000,000"),
    Map.entry("options", "2.0 Engine, SunRoof")
);

System.out.println("fuel : " + carInfo.getOrDefault("fuel", "gasoline"));// fuel : gasoline
```

- 계산 패턴
    - 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황 구분 가능.
    - `computeIfAbsent` : 제공된 키에 해당하는 값이 없으면(값이 없거나 null), 키를 이용해 연산 후 맵에 추가.
    - `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가.
    - `compute` : 제공된 키로 새 값을 계산하고 맵에 저장.

```java
Map<Integer, Integer> square = new HashMap<>() {{
    put(1, 1);
    put(2, 0);
}};

square.computeIfAbsent(3, (key) -> key * key);// key3의 값(value)이 없으므로 연산 후 추가 {3 : 9}
square.computeIfAbsent(3, (key) -> key * key * key);// key3의 값(value)이 존재하므로 추가되지 않음
square.computeIfPresent(4, (key, value) -> key * key);// key4가 존재하지 않아 추가되지 않음
square.compute(2, (key, value) -> key * key);// key2의 값(value) 연산 후 추가

System.out.println("square : " + square);// square : {1=1, 2=4, 3=9}

/* Map<K, List<V>> 같은 경우 요소를 추가할때 computeIfAbsent를 사용가능 */
Map<String, List<String>> carsOfBrand = new HashMap<>();
// Kia 값이 없으면 new ArrayList<>() 반환, 있으면 기존 값 반환
carsOfBrand.computeIfAbsent("Kia", brand -> new ArrayList<>()).add("K5");
```

- 삭제 패턴
    - 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공.

```java
map.remove(key, value);
```

- 교체 패턴
    - `replaceAll` : BiFunction을 적용한 결과로 각 항목의 값을 교체.
    - `replace` ****: 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값과 연관되었을 때만 값을 교체하는 overload 버전.

```java
Map<String, Integer> numbers = new HashMap<>() {{
    put("number1", 1);
    put("number2", 2);
    put("number3", 3);
}};

numbers.replaceAll((key, value) -> value + 10);
System.out.println("numbers : " + numbers);// numbers : {number1=11, number2=12, number3=13}

numbers.replace("number1", 1);// if (key == "number1") value = 1;
System.out.println("numbers : " + numbers);// numbers : {number1=1, number2=12, number3=13}

numbers.replace("number2", 2, 2);// if (key == "number2" && value == 2) value = 2;
System.out.println("numbers : " + numbers);// numbers : {number1=1, number2=12, number3=13}
```

- 합침
    - 두 맵을 합쳐야 할때 forEach와 merge 메서드를 이용해 충돌없이 해결 가능.

```java
Map<String, Integer> numbers1 = new HashMap<>() {{
    put("number1", 10);
    put("number2", 20);
    put("number3", 30);
    put("number4", 40);
}};

Map<String, Integer> numbers2 = new HashMap<>() {{
    put("number1", 1);
    put("number2", 2);
    put("number3", 3);
}};

numbers1.forEach((k, v) -> numbers2.merge(k, v, Integer::sum));
System.out.println("numbers2 : " + numbers2);
// numbers2 : {number1=11, number2=22, number3=33, number4=40}
```

---

### 1.7 개선된 ConcurrentMap

- ConcurrentHashMap 클래스 :
    - 동시성 친화적이며 최신 기술을 반영한 HashMap 버전.
    - 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용.
    - 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다( 표준 HashMap은 비동기로 동작 ).

- 리듀스와 검색
    - `forEach` : 각 (키, 값) 쌍에 주어진 액션을 실행.
    - `reduce` : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 함침.
    - `search` : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용.
- 키에 함수 받기, 값, Map.Entry, (키, 값) 인수를 이용한 네 가지 연산 형태를 지원.
    - 키, 값으로 연산 : `forEach`, `reduce`, `search`.
    - 키로 연산 : `forEachKey`, `reduceKey`, `searchKey`.
    - 값으로 연산 : `forEachValue`, `reduceValues`, `searchValues`.
    - Map.Entry 객체로 연산 : `forEachEntry`, `reduceEntries`, `searchEntries`.
- 특징 : 상태를 잠그지 않고 연산 수행.
    - 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 X.
    - 연산에 병렬성 기준값을 지정. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행.
    - 기준값을 1로 지정하며 공통 스레드 풀을 이용해 병렬성을 극대화.
    - Long.MAX_VALUE를 기준값으로 설정하면 한 개의 스레드로 연산을 실행.
    - 소프트웨어 아키텍처가 고급 수준의 자원 활용 최적화를 사용하고 있지 않다면 기준값 규칙을 따르는 것 권장.
    - int, long, double 등의 기본값에는 전용 each reduce 연산이 제공되므로 reduceValuesToInt, reduceKeysToLong 등을 이용하면 박싱 작업을 할 필요가 없고 효율적으로 작업을 처리가능.

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Long> maxValue = Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

- 계수
    - `mappingCount()` : 맵의 매핑 개수 반환.
    - size 메서드 대신 새 코드에서는 int를 반환하는 mappingCount 메서드 사용 권장 ⇒ 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처 가능.
- 집합뷰
    - `keySet()` : ConcurrentHashMap을 집합 뷰로 반환
    - 맵을 바꾸면 집합도 바뀌고 반대로 맵도 영향 존재 ⇒ newKeySet을 이용해 ConcurrentHashMap으로 유지되는 집합을 생성가능.

---

### 2. 리팩터링, 테스팅, 디버깅

### 2.1 리팩터링

- **익명 클래스 → 람다 표현식**
    - 익명클래스 : 코드를 장황하게 만들고, 쉽게 에러를 일으킴.
    - 람다 : 간결, 가독성이 좋음.
    - 익명클래스를 람다 표현식으로 변환할 수 없는 경우 :
        - **super, this** : 익명 클래스의 this는 자기 자신을 의미, 람다의 this는 람다를 감싸는 클래스를 의미( super도 마찬가지).
        - **섀도우 변수** : 익명클래스는 섀도우변수 O, 람다는 섀도우 변수 X.
        - **콘텍스트 오버로딩에 따른 모호함** : 익명클래스는 인스턴스화시 형식이 정해짐, 람다는 콘텍스트에 따라 달라딤.

```java
/* 익명 클래스 */
Runnable runnable = new Runnable() {
	public void run() {
		System.out.println("Hello");
	}
}
/* 람다 표현식 */
Runnable runnable = () -> System.out.println("Hello");

/* 섀도우 변수 사용 */
int num = 10;
Runnable runnable = new Runnable() {
	public void run() {
		int num = 11; // 정상작동
		System.out.println(num);
	}
}
Runnable runnable = () -> {
	int num = 11; // 컴파일에러
	System.out.println(num);
}

/* 콘텍스트 오버로딩에 따른 모호함 */
interface Task {
	public void execute();
}
public static void doSomething(Runnable r) { r.run(); }
public static void doSomething(Task r) { r.execute(); }

doSomething(new Task() {
	public execute() {
		System.out.println("Dager danger!!");
	}
});
doSomething(() -> System.out.println("Danger danger!!")); // Runnable과 Task 중 어떤 것을 의미하는지 모호함
/* 명시적 형변환을 통해 모호함 제거 가능 */
doSomething((Task)() -> System.out.println("Danger danger!!"));
```

- **람다 표현식 → 메서드 참조**
    - 메서드 참조 : 람다 표현식에 비해 가독성이 더 좋음.
    - 메서드 참조 & 정적 헬퍼 메서드( Collectors API ) : 코드의 의도가 더욱 명확해짐.

```java
/* 람다 표현식 */
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
			.collect(groupingBy(dish -> {
				if(dish.getCalories() <= 400) return CaloricLevel.DIET;
				else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
				else return CaloricLevel.FAT;
}));

/* 메서드 참조 : 람다 표현식을 메서드로 추출 */
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(Dish::getCaloricLeve));
/* 메서드 참조 : 정적 헬퍼 메서드 활용 */
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

- **조건부 연기 실행**
    - 캡슐화 강화 : 클라이언트 코드로 객체 상태 노출 X.
    - 동작 실행 연기 : 특정 상황을 확인( 동작실행 트리거 )하는 메서드를 인자로 활용.
    - 코드 가독성 up.

```java
if(logger.isLoggable(Log.FINER)) {
	logger.finer("problem: " + generateDiagnostic());
}

/* 조건부 연기 실행 */
public void log(Level level, Supplier<String> msgSupplier) {
	if(logger.isLoggable(level)) {
		log(level, msgSupplier.get());
	}
}
logger.log(Level.FINER, () -> "problem: " + generateDiagnostic());
```

- **실행 어라운드**
    - 매번 같은 준비, 종료과정을 반복적으로 수행하는 코드 → 람다로 사용( 코드 중복 down ).
    - 람다를 활용해서 다양한 방식으로 파일을 처리할 수 있도록 파라미터화.

---

### 2.2 람다로 객체지향 디자인 패턴 리팩터링

- 람다를 이용하면 이전에 디자인 패턴으로 해결하던 문제를 더 쉽고 간단하게 해결 가능.
- 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재구현 가능.

- **전략**
    - 전략 패턴 : 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법.
    - 구성 요소 :
        - 알고리즘을 나타내는 인터페이스( Streategy 인터페이스 ).
        - 알고리즘을 구현하는 클래스( Strategy implements ).
        - 해당 알고리즘을 사용하려하는 클라이언트.
    - 람다 + 전략 패턴 : 코드 간결화 & 캡슐화.

    ```java
    /* 전략 패턴 : 알고리즘 정의 */
    public interface ValidateStreategy {
    	boolean execute(String s);
    }
    /* 전략 패턴 : 알고리즘 구현 */
    public class IsAllLowerCase implements ValidateStreategy {
    	public boolean exceute(String s) {
    		return s.matches("[a-z]+");
    	}
    }
    public class IsNumeric implements ValidateStreategy {
    	public boolean execute(String s) {
    		return s.matches("\\d+");
    	}
    }
    public class Validator {
    	private final ValidationStreategy strategy;
    	public Validator(ValidationStreategy v) {
    		this.strategy = v;
    	}
    	public boolean validate(String s) {
    		return strategy.execute(s);
    	}
    }
    
    /* 클라이언트 코드 : 기존 */
    Validator numericValidator = new Validator(new IsNumeric());
    boolean isNumeric = numericValidator.validate("aaaa");
    Validator lowerValidator = new Validator(new IsAllLowerCase());
    boolean isLower = lowerValidator.validate("bbbb");
    
    /* 클라이언트 코드 : 람다 -> 알고리즘 구현부 자체를 대체 가능 */
    Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
    boolean isNumeric = numericValidator.validate("aaaa");
    Validator lowerValidator = new Validator((String s)) -> s.matches("\\d+"));
    boolean isLower = lowerValidator.validate("bbbb");
    ```

- **템플릿 메서드**
    - 템플릿 메서드 패턴 : 이 알고리즘을 사용하고 싶은데, 그대로는 안되고 조금 고쳐야 하는 상황.
    - 구성요소 :
        - 알고리즘을 추상적으로 표현한 메서드.
        - 해당 알고리즘을 수정, 추가하기 위한 구현 메서드.
    - 람다 + 템플릿 메서드 패턴 : 정의된 개요에 추가적 기능을 간결하게 작성 가능.

    ```java
    /* 공통 알고리즘 정의 */
    abstract class OnlineBanking {
    	public void processCustomer(int id) {
    		Customer c = Database.getCustomerWithId(id);
    		makeCustomerHappy(c);
    	}
    	abstract void makeCustomerHappy(Customer c);
    }
    
    /* 람다 표현식 : 이전에 정의한 makeCustomerHappy의 메서드 시그니처와 일치하도록 Consumer 형식을 갖는 두 번째 인수를 메서드에 추가 */
    public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    	Customer c = Database.getCustomerWithId(id);
    	makeCustomerHappy.accept(c);
    }
    new OnlineBankingLambda().processCustomer(1337, (Customer c) -> print("hello" + c.getName()));
    ```

    - **옵저버**
        - 옵저버 패턴 : 어떤 이벤트가 발생했을 때 한 객체(subject)가 다른 객체 리스트(observer)에 자동으로 알림을 보내야 하는 상황.
        - 구성 요소 :
            - 알림을 보내야 하는 이벤트.
            - 알림을 보내는 동작 추상화.
            - 알림을 보내는 동작 구체화.
        - 람다 + 옵저버 패턴 :
            - Observer 인테페이스를 구현하는 클래스를 만드는 대신 람다 표현식을 직접 전달해서 실행할 동작을 지정 가능.
            - 하지만 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현방식을 고수하는 것이 바람직.

    ```java
    /* 알림을 보내는 동작 추상화 */
    interface Observer {
      void notify(String tweet);
    }
    /* 알림을 보내는 동작 구체화 */
    public class NyTimes implements NotiObserver {
        @Override
        public void notify(String tweet) {
            if (tweet != null && tweet.contains("monety")) {
                System.out.println("Breaking news in NY ! " + tweet);
            }
        }
    }
    public class Guardian implements NotiObserver {
        @Override
        public void notify(String tweet) {
            if (tweet != null && tweet.contains("queen")) {
                System.out.println("Yet more new from London .. " + tweet);
            }
        }
    }
    public class LeMonde implements NotiObserver {
        @Override
        public void notify(String tweet) {
            if (tweet != null && tweet.contains("wine")) {
                System.out.println("Today cheese, wine and news! " + tweet);
            }
        }
    }
    /* 알림을 보내야 하는 상황 */
    public interface NotiSubject {
        void registerObserver(NotiObserver o);
        void notifyObservers(String tweet);
    }
    public class Feed implements NotiSubject {
        private final List<NotiObserver> observers = new ArrayList<>();
        @Override
        public void registerObserver(NotiObserver o) {
            observers.add(o);
        }
    
        @Override
        public void notifyObservers(String tweet) {
            observers.forEach(o -> o.notify(tweet));
        }
    }
    
    Feed f = new Feed();
    f.registerObserver(new NyTimes());
    f.registerObserver(new LeMonde());
    f.registerObserver(new Guardian());
    f.notifyObservers("The Queen ...")
    
    /* 람다 */
    Feed feed = new Feed();
    feed.registerObserver((String tweet) -> {
        if(tweet != null && tweet.contains("money")) {
            System.out.println("Breaking news in NY ! " + tweet);
        }
    });
    feed.registerObserver((String tweet) -> {
        if(tweet != null && tweet.contains("queen")) {
            System.out.println("Yet more new from London .." + tweet);
        }
    });
    ```

- **의무 체인**
    - 의무 체인 패턴 : 작업 처리 객체의 체인(동작 체인 등)을 만들 때.
        - 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식.
    - 일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성.
        - 작업 처리 객체가 자신의 작업을 끝냈으면 다음 작업 처리 객체로 결과를 전달.
    - 람다 + 의무 체인 패턴 :
        - 함수 체인 방식과 유사.
        - 람다 표현식을 조합하는 방식으로는 기본적으로 compose, andThen 존재.
        - andThen메서드로 이들 함수를 조합해 체인을 생성.

    ```java
    /* 작업 처리 코드 */
    public abstract class ProcessingObject<T> {
    	protected ProcessingObject<T> successor;
    	public void setSuccessor(ProcessingObject<T> successor) {
    		this.successor = successor;
    	}
    
    	public T handle(T input) {
    		T r = handleWork(input);
    		if (successor != null) {
    			return successor.handle(r);
    		}
    		return r;
    	}
    	
    	abstract protected T handleWork(T input);
    }
    
    /* 기존 작업 처리 패턴 */
    public class HandleTextProcessing extends ProcessingObject<String> {
        @Override
        protected String hadleWork(String input) {
            return "From Raoul, Mario and Alan : " + input;
        }
    }
    public class SpellCheckProcessing extends ProcessingObject<String> {
        @Override
        protected String hadleWork(String input) {
            return input.replaceAll("labda", "lambda");
        }
    }
    ProcessingObject<String> p1 = new HandleTextProcessing();
    ProcessingObject<String> p2 = new SpellCheckProcessing();
    p1.setSuccessor(p2);
    p1.handle("Aren't ladbas really sexy?");
    
    /* 람다 */
    UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan : " + text;
    UnaryOperator<String> spellCheckProcessing = (String text) -> text.replaceAll("labda", "lambda");
    Function<String, String> pipeline = headerProcessing.andThen(spellCheckProcessing);
    pipeline.apply("Aren't ladbas really sexy?");
    ```

- **팩토리**
    - 팩토리 패턴 : 인스턴스화 로직을 클라이언트에게 노출하지 않고 객체를 만들 때.
    - 람다 + 팩토리 패턴 :
        - 팩토리 메서드 createProduct가 상품 생성자로 여러 인수로 전달하는 상황에서는 인수 개수에 맞게 특별한 함수형 인터페이스를 만들어야 함 → Map 시그니처가 복잡해짐.

    ```java
    /* 기존 코드 */
    public class ProductFactory {
        public static Product createProduct(String name) {
            switch (name) {
                case "loan" : return new Loan();
                case "stock" : return new Stock();
                case "bond" : return new Bond();
                default: throw new RuntimeException("...");
            }
        }
    }
    
    Product p = ProductFactory.createProduct("loan");
    
    /* 람다 */
    final static Map<String, Supplier<Product>> map = new HashMap<>();
    static {
      map.put("loan", Loan::new);
      map.put("stock", Stock::new);
      map.put("bond", Bond::new);
    }
    
    public static Product createProduct(String name) {
      Supplier<Product> p = map.get(name);
      if (p != null) {
        return p.get();
      }
      throw new IllegalArgumentException("No such product: " + name);
      ...
    }
    ```


---

### 3. 테스팅

- 람다에 대해서도 단위 테스팅(unit testing)이 작성되어야만 함.
- 하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 X.

- **보이는 람다 표현식의 동작 테스팅**
    - 람다의 동작을 테스트 하기 위해 람다를 필드에 저장해서 테스트 가능.
- **람다를 사용하는 메서드의 동작에 집중**
    - 람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화 하는것.
    - 람다 표현식을 사용하는 메서드의 동작을 테스트함으로서 람다 표현식을 검증 할 수 있음.
- **복잡한 람다를 개별 메서드로 분할**
    - 복잡한 로직이 포함된 람다를 구현하게 된다면 로직을 분리 하거나 메서드 레퍼런스를 활용할 것.
    - 그러면 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있음.
- **고차원 함수 테스팅**
    - 고차원 함수 : 함수를 인수로 받거나 다른 함수를 반환하는 메서드.
    - 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트 할 수 있음.
    - 테스트해야하는 함수가 다른 함수를 반환한다면 함수형 인터페이스의 인스턴스로 간주하고 테스트 할 수 있음.

---

### 4. 디버깅

- 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화함.

- **스택 트레이스 확인**
    - 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성됨.
    - 그렇기에 람다 표현식과 관련된 스택 트레이스는 이해하기 어려울 수 있음 ⇒ 자바 컴파일러가 개선해야할 부분.
- **정보 로깅**
    - forEach를 통해 스트림 결과를 출력하거나 로깅할 수 있음. 하지만 forEach는 스트림을 소비하는 연산.
    - 스트림 파이프라인에 적용된 각각의 연산의 결과를 확인할 수 있다면 대신 peek라는 스트림 연산을 활용할 수 있음.
    - peek는 스트림의 각 요소를 소비한것 처럼 동작을 실행하지만, 실제로 스트림을 소비하지않고 자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달.

---

### 5. 람다를 이용한 도메인 전용 언어

- 도메인 전문가는 소프트웨어 개발 프로세스에 참여할 수 있고, 비즈니스 관점에서 소프트웨어가 제대로 되었는지 확인할 수 있음.
- 결과적으로 버그와 오해를 미리 방지 가능.
- 도메인 전용 언어(DSL)로 애플리케이션 비즈니스 로직을 표현함으로써 해당 문제 해결 가능.

- DSL : 특정 도메인을 인터페이스로 만든 API.
- DSL 장점 :
    - **간결함** : 비즈니스 로직을 간편하게 캠슐화 ⇒ 반복X, 코드 간결.
    - **가독성** : 도메인 영역의 용어 사용 ⇒ 다양한 조직 구성원 간에 코드와 도메인 영역 공유.
    - **유지보수** : 잘 설계된 DSL을 통해 구현된 코드는 쉽게 수정, 유지보수 가능.
    - **높은 수준의 추상화** : 저수준의 구현 세부 사항 메서드는 클래스의 비공개로 만들어서 숨길 수 있음.
    - **집중** : 프로그래머가 특정 코드에 집중 가능.
    - **관심사분리** : 독립적으로 비즈니스 관련된 코드에서 집중이 용이 ⇒ 유지보수 쉬워짐.
- DSL 단점 :
    - **DSL 설계의 어려움** : 간결하게 제한적인 언어에 도메인 지식을 담는 것의 어려움.
    - **개발 비용** : 초기 프로젝트에 많은 비용과 시간이 소모됨.
    - **추가 우회 계층** : DSL은 추가적인 계층으로 도메인 모델을 감싸며, 이 때 계층을 최대한 작게 만들어 성능 문제 회피.
    - **새로 배워야 하는 언어** : DSL을 위한 언어를 하나더 배워야 함.
    - **호스팅 언어 한계** : 일부 범용 프로그래밍언어의 장황함과 엄격한 문법으로 DSL을 만들기가 힘듦.
        - 자바의 람다가 해당 문제를 해결 가능.

---

### 5.1 내부 DSL

- 내부 DSL : 순수 자바코드 같은 기존 호스팅 언어를 기반으로 구현한 DSL.
- 유연성이 떨어지기 때문에 간단하고 표현력 있는 DSL을 만드는데 한계가 있었지만 람다 표현식이 등장하면서 어느정도 해결될 수 있음
- 순수 자바로 DSL 을 구현하면 다음과 같은 장점이 존재.
    - 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 감소.
    - 순수 자바로 DSL 을 구현하면 나머지 코드와 함께 컴파일 가능( 다른 언어의 컴파일러 또는 외부 DSL을 만드는 도구를 사용하지 않아도 됨 ).
    - 새로운 언어를 배우거나 복잡한 외부 도구를 배울 필요가 없음.
    - DSL 사용자는 기존의 자바 IDE로 자동 완성, 리팩터링 기능 그대로 사용 가능.
    - 추가 DSL을 쉽게 합칠 수 있음.

---

### 5.2 다중 DSL

- 다중 DSL : 자바는 아니지만 JVM에서 실행되며 더 유연하고 표현력이 가능한 언어(ex. 스칼라, 그루비…)로 구현한 DSL.
- 코틀린, 실론 같이 스칼라와 호환성이 유지되며 단순하고 쉽게 배울 수 있는 새 언어도 존재.
- 이 언어들은 모두 자바보다 젋으며 제약을 줄이고, 간편한 문법을 지향하도록 설계.
- DSL 친화적이지만 다음과 같은 단점들이 존재한다.
    - 새로운 프로그래밍 언어를 배우거나 누군가 해당 기술을 지니고 있어야 함.
    - 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야 함.
    - JVM 에서 실행되지만 자바와 호환성이 완벽하지 않은 경우가 많음 (성능 손실 가능).

---

### 5.3 외부 DSL

- 외부 DSL : 스탠드어론(standalone) 이라고 불리우며 호스팅 언어와는 독립적으로 자체의 문법을 가지는 DSL.
- 자신만의 문법과 구문으로 새 언어를 설계해야 하고 언어 파싱, 파서의 결과 분석, 외부 DSL 실행할 코드를 만들어야 함.
- 이 방법을 선택해야 한다면 ANTLR 같은 자바 기반 파서 생성기를 이용하면 도움을 받을 수 있음.
- 장점
    - 무한한 유연성을 가짐.
    - 필요한 특성을 완벽하게 제공하는 언어로 설계 가능.
    - 비즈니스 문제를 묘사하고 해결하는 가독성 좋은 언어 선택 가능.
    - 자바로 개발된 인프라구조 코드와 비즈니스 코드를 명확하게 분리 가능.
- 단점
    - 일반적인 작업이 아니며 쉽게 기술을 얻을 수 없음.
    - 작업이 복잡하고 제어 범위를 쉽게 벗어날 수 있음.
    - 처음 설계한 목적을 벗어나는 경우가 많음.
    - DSL과 호스트 언어 사이에 인공 계층이 생김.

---

### 5.4 **스트림 API는 컬렉션을 조작하는 DSL**

- Stream 인터페이스는 네이티브 자바 API 에 작은 내부 DSL을 적용한 좋은 예.
- 컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지만 강력한 DSL.
- Stream 인터페이스를 이용하여 함수형으로 구현하면 쉽고 간결.
- 스트림 API의 플루언트 형식 또한 설계된 DSL의 특징 중 하나( 중간 연산 게으름, 다른 연산으로 파이프라인 가능 ).

---

### 5.5 **데이터를 수집하는 DSL인 Collectors**

- Collector 인터페이스는 데이터 수집(수집, 그룹화, 파이션)을 수행하는 DSL로 간주 가능.
- 특히 다중 필드 정렬을 지원하도록 합쳐질 수 있으며, `Collectors` 는 다중 수준 그룹화를 달성할 수 있도록 합쳐질 수 있음.

---

### 5.6 **자바로 DSL을 만드는 패턴과 기법**

```java
public class Stock {
    private String symbol;
    private String market;
    public String getSymbol(){
        return symbol;
    }
    public void setSymbol(String symbol) {
        this.symbol = symbol;
    }
    ...
}
public class Trade {
    public enum Type { BUY, SELL }
    
    private Type type;
    private Stock stock;
    private int quantity;
    private double price;
    ...    
}

public class Order {
    private String customer;
    private List<Trade> trades = new ArrayList<>();
    ...
}

/* 주문 생성 코드 : 장황하고 비개발자인 도메인 전문가가 이해하고 검증하기 어려움 */
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.Buy);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

trade.setStock(stock1);
...
```

- **메서드 체인** : 메서드 체인은 DSL 에서 가장 흔한 방식 중 하나.
- 장점 :
    - 사용자가 지정된 절차에 따라 플루언트 API의 메서드 호출을 강제.
    - 파라미터가 빌더 내부로 국한 됨.
    - 메서드 이름이 인수의 이름을 대신하여 가독성 개선.
    - 문법적 잡음이 최소화.
    - 선택형 파라미터와 잘 동작.
    - 정적 메서드 사용을 최소화하거나 없앨 수 있음.
- 단점 :
    - 빌더를 구현해야 함.
    - 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요.
    - 도메인 객체의 중첩 구조와 일치하게 들여쓰기를 강제할 수 없음.

```java
/* 메서드 체인 : 빌더 구현 */
public class MethodChainingOrderBuilder {

    public final Order order = new Order();
    
    private MethodChainingOrderBuilder(String customer) {
        order.setCustomer(customer);
    }
    public static MethodChainingOrderBuilder forCustomer(String customer) {
        return new MethodChainingOrderBuilder(customer);
    } 
    public TradeBuilder buy(int quantity) {
        return new TradeBuilder(this, Trade.Type.SELL, quantity);
    }
    public MethodChainingOrderBuilder addTrade(Trade trade) {
        order.addTrade(trade);
        return this;
    }
    public Order end() {
        return order;
    }
}

public class TradeBuilder {
    private final MethodChainingOrderBuilder builder;
    public final Trade trade = new Trade();
    
    private TradeBuilder(MethodChainingOrderBuilder builder, Trade.Type type, int quantity) {
        this.builder = builder;
        trade.setType(type);
        trade.setQuantity(quantity);
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(builder, trade, symbol);
    }
}

public class StockBuilder {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    private StockBuilder(MethodChainingOrderBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    } 
    public TradeBuilderWithStock on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return new TradeBuilderWithStock(builder, trade);
    }    
}

public class TradeBuilderWithStock {
    private final MethodChainingOrderBuilder builder;
    private final Trade trade;
    
    public TradeBuilderWithStock(MethodChainingOrderBuilder builder, Trade trade) {
        this.builder = builder;
        this.trade = trade;
    }
    public MethodChainingOrderBuilder at(double price) {
        trade.setPrice(price);
        return builder.addTrade(trade);
    }
}

/* 메서드 체인 : DSl */
Order order = forCustomer("BigBank")
    .buy(80)
    .stock("IBM")
    .on("NYSE")
    ...
    .end();
```

- **중첩된 함수 이용** : 다른 함수 안에 함수를 이용해 도메인 모델 생성.
- 장점 :
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영구현의 장황함을 줄일 수 있음.
- 단점
    - 결과 DSL 에 더 많은 괄호를 사용.
    - 정적 메서드 사용이 빈번.
    - 인수 목록을 정적 메서드에 넘겨줘야 함.
    - 인수의 의미가 이름이 아니라 위치에 의해 정의 됨.
    - 도메인에 선택 사항 필드가 있으면 인수를 생략할 수 있으므로 메서드 오버로딩 필요.

```java
/* 중첩된 함수 이용 */
public class NestedFunctionOrderBuilder {

    public static Order order(String customer, Trade... trades) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }
    public static Trade buy(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.BUY);
    }    
    private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type type) {
        Trade trade = new Trade();
        trade.setQuantity(quantity);
        trade.setType(type);
        trade.setStock(stock);
        trade.setPrice(price);
        return trade;
    }
    public static double at(double price) {
        return price;
    }
    publid static Stock stock(String Symbol, String market) {
        Stock stock = new Stock();
        stock.setSymbol(symbol);
        stock.setMarket(market);
        return stock;
    }
}

Order order = order("BigBank",
                    buy(80, stock("IBM", on("NYSE")), at(125.00)),
                    sell(50, stock("GOOGLE", on("NASDAQ")), at(375.00))
                    );
```

- 람다 표현식을 이용한 시퀀싱 : 람다 표현식으로 정의한 함수 시퀀스를 사용.
- 장점
    - 플루언트 방식으로 도메인 객체 정의 가능.
    - 중첩 방식이 도메인 객체 계층 구조에 그대로 반영.
    - 선택형 파라미터와 잘 동작.
    - 정적 메서드를 최소화하거나 없앨 수 있음.
    - 빌더의 접착 코드가 없음.
- 단점
    - 설정 코드가 필요.
    - 람다 표현식 문법에 의한 잡음의 영향을 받음.

```java
public class LambdaOrderBuilder {

    private Order order = new Order();
    
    public static Order order(Consumer<LambdaOrderBuilder> consumer) {
        LambdaOrderBuilder builder = new LambdaOrderBuilder();
        consumer.accept(builder);
        return builder.order;
    }
    public void forCustomer(String customer) {
        order.setCustomer(customer);
    }
    public void buy(Consumer<TradeBuilder> consumer) {
        trade(consumer, Trade.Type.BUY);
    }
    private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(type);
        consumer.accept(builder);
        order.addTrade(builder.trade);
    }
}

public class TradeBuilder {

    private Trade trade = new Trade();
    
    public void quantity(int quantity) {
        trade.setQuantity(quantity);
    }
    public void price(double price) {
        trade.setPrice(price);
    }
    public void stock(Consumer<StockBuilder> consumer) {
        StockBuilder builder = new StockBuilder();
        consumer.accept(builder);
        trade.setStock(builder.stock);
    }
}
...

Order order = order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        });
    });
})
```

- **조합하기** : 중첩된 함수 패턴과 람다 기법을 혼용.
- 장점
    - 가독성 향상
- 단점
    - 여러 기법을 혼용했기 때문에 사용자가 DSL을 배우는 시간이 오래 걸림

```java
public class MixedBuilder {

    public static Order forCustomer(String customer, TradeBuilder... builders) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(builders).forEach(b -> order.addTrade(b.trade));
        return order;
    } 
    public static TradeBuilder buy(Consumer<TradeBuilder> consumer) {
        return builderTrade(consumer, Trade.Type.BUY);
    }
    private static TradeBuilder buildTrade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(buy);
        consumer.accept(builder);
        return builder;
    }
}

public class TradeBuilder {
    
    private Trade trade = new Trade();
    
    public TradeBuilder quantity(int quantity) {
        trade.setQuantity(quantity);
        return this;
    }
    public TradeBuilder at(double price) {
        trade.setPrice(price);
        return this;
    }
    public StockBuilder stock(String symbol) {
        return new StockBuilder(this, trade, symbol);
    }
}

public class StockBuilder {
    
    private final TradeBuilder builder;
    private final Trade trade;
    private final Stock stock = new Stock();
    
    public StockBuilder(TradeBuilder builder, Trade trade, String symbol) {
        this.builder = builder;
        this.trade = trade;
        stock.setSymbol(symbol);
    }
    public TradeBuilder on(String market) {
        stock.setMarket(market);
        trade.setStock(stock);
        return builder;
    }
}

Order order = 
    forCustomer("BigBank", 
                buy(t -> t.quantity(80)
                          .stock("IBM")
                          .on("NYSE")
                          .at(125.00)),
                sell(t -> t.quantity(50)
                           .stock("GOOGLE")
                           .on("NASDAQ")
                           .at(125.00)));
```

- **DSL에 메서드 참조 사용하기** : 간결하고 유연한 방식의 구현 가능.

---

### 6. 실생활의 자바 8 DSL

### 6.1 **jOOQ**

- jOOQ는 SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어.
- jOOQ DSL 구현하는 데에는 메서드 체인 패턴을 사용.
- SQL 문법을 흉내내려면 선택적 파라미터를 허용하고 정해진 순서대로 특정 메서드가 호출되어야 하기 때문에 메서드 체인 패턴의 특성이 필요.

---

### 6.2 **큐컴버**

- 동작 주도 개발(BDD)은 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 도메인 전용 스크립팅 언어를 사용.
- 큐컴버(cucumber)는 BDD 프레임워크로 명령문을 실행할 수 있는 테스트 케이스로 변환하며, 비즈니스 시나리오를 평문으로 구현할 수 있도록 도와줌.
- 큐컴버는 세가지 구분되는 개념을 사용.
    - Given: 전제 조건 정의.
    - When: 시험하려는 도메인 객체의 실질 호출.
    - Then: 테스트 케이스의 결과를 확인.

---

### 6.3 **스트링 통합**

- 스프링 통합은 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장.
- 복잡한 통합 솔루션 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있도록 돕는 것이 스프링 통합의 핵심 목표.
- 스프링 통합 DSL 에서도 메서드 체인 패턴이 가장 널리 사용되고 있음.

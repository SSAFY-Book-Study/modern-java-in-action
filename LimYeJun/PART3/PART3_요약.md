## Chapter8 컬렉션 API 개선

### 컬렉션 팩토리

#### 리스트 팩토리
자바에서 적은 요소를 포함하는 리스트를 만드는 방법?
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```

`Arrays.asList()` 팩토리 메서드를 사용하면 코드를 간단하게 줄일 수 있음
```java
List<String> friends
    = Arrays.asList("Raphael", "Olivia", "Thibaut");
```

하지만 위의 방식은 고정 크기의 리스트를 만드는 팩토리 메서드로 `요소를 갱신할 순 있지만 요소를 추가하면 에러 발생`
```java
List<String> friends
    = Arrays.asList("Raphael", "Olivia");
friends.set(0, "Richard");
friends.add("Thibaut");    // UnsupportedOperationException
```
`List.of()` 팩토리 메서드를 이용하면 set 메서드로 아이템을 바꾸려해도 비슷한 예외가 발생한다 (불변성)

#### 집합 팩토리
`List.of()`와 비슷한 방법으로 바꿀 수 없는 집합을 만들 수 있다
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
중복된 요소를 제공해 집합을 만들려고하면 `IllegalArgumentException`이 발생한다
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");  // error
```

#### 맵 팩토리
맵을 만드는 것은 리스트나 집합을 만드는 것에 비해 조금 복잡하다

자바9에서는 두 가지 방법으로 바꿀 수 없는 맵을 초기화할 수 있다
```java
Map<String, Integer> ageOfFriends
  = Map.of("Raphael", 30, "Olivia", 25, "Thibuat", 26);
```
10개 이하의 키와 값 쌍에서는 `Map.of`가 유리
그 이상의 맵에서는 `Map.Entry`객체를 인수로 받는 `Map.ofEntires` 팩토리 메서드를 이용하는 것이 좋다

```java
Map<String, Integer> ageOfFriends
  = Map.ofEntries(entry("Raphael", 30),
                  entry("Olivia", 25),
                  entry( "Thibuat", 26));
```

### 리스트와 집합 처리

- `removeIf`: 프레디케이트를 만족하는 요소를 제거
- `replaceAll`: `UnaryOperator` 함수를 이용해 요소를 바꾼다
- `sort`: 리스트 정렬

#### removeIf 메서드

숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드
```java
for (Transaction transaction : transactions) {
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
        }
    }
```
위 코드는 `ConcurrentModificationException`을 일으킨다
왜? -> 내부적으로 for-each 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석 됨

```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
        }
    }
```
- 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않는다
- Iterator 객체를 명시적으로 사용하고 그 객체의 remove()를 호출함으로 이 문제를 해결할 수 있음

`해결`
```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
            iterator.remove();
        }
    }
```
이를 자바8 이후의 코드로 단순하게 바꿀 수 있다

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

`replaceAll`도 마찬가지다 기존의 `stream().map`을 통해 리스트의 각 요소를 새로운 요소로 바꿀 수 있지만 **새 문자열 컬렉션**을 만드는 것이다
Iterator.set 메서드를 이용하면 기존 컬렉션을 바꾸는 것이 가능하며 이를 `replaceAll`로 간단하게 구현할 수 있다

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

### 맵 처리

자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다
자주 사용되는 패턴을 개발자가 직접 구현할 필요가 없도록 이들 메서드를 추가한 것

#### forEach 메서드

`Map.Entry(K, V)`의 반복자를 이용해 맵의 항목 집합을 반복할 수 있다

```java
for(Map.Entry(String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
```
자바 8부터 Map 인터페이스는 BiConsumer(키와 값을 인수로 받음)를 인수로 받는 forEach 메서드를 지원하므로 코드를 간결하게 할 수 있다
```java
ageOfFirends.forEach((friend, age) -> System.out.println(frined + " is " + age + " years old"));
```

#### 정렬 메서드
두 개의 유틸리티를 이용하면 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다

- Entry.comparingByValue
- Entry.comparingByKey

```java
ageOfFirends
      .entrySet()
      .stream()
      .sorted(Entry.comparingByKey())
      .forEachOrdered(System.out::println);
```

#### HashMap 성능
자바 8에서 HashMap의 내부 구조를 바꿔 성능을 개선했다
기존 맵의 항목 키로 해시코드로 접근할 수 있는 버킷에 저장했는데, 최악의 상황에는 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환해야 하므로 성능이 저하된다
따라서 O(log(n))이 걸리는 정렬된 트리구조로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다 
**키가 String, Number 같은 Comparable의 형태여야만 정렬된 트리가 지원됨**

#### getOrDefault 메서드
기존에는 찾으려는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 요청 결과가 널인지 확인해야 한다
- 키가 존재하더라도 값이 널인 상황에선 getOrDefault가 널을 반환할 수 있다

#### 계산 패턴
- computeIfAbsent: 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가
- computeIfPresent: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
- compute: 제공된 키로 새 값을 계산하고 맵에 저장

키에 해당하는 값이 있거나 없을 때 필요한 계산을 전달할 수 있음

#### 삭제 패턴
제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드는 이미 알고 있다. 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다 
`favouriteMovies.remove(key, value);`

#### 교체 패턴
맵의 항목을 바꾸든 데 사용할 수 있는 두 개의 메서드가 추가되었다
- `replaceAll`: BiFunction을 적용한 결과로 각 항목의 값을 교체한다 -> List의 replaceAll과 비슷한 동작을 수행
- `replace`: 키가 존재하면 맵의 값을 바꾼다 -> 키가 특정 값으로 매핑되었을 때만 교체하는 오버로드 버전도 있다

#### 합침
두 그룹의 연락처를 포함하는 두 개의 맵을 합친다고 가정하면 `putAll`을 사용할 수 있다
```java
Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```
중복된 키가 없다면 잘 동작한다
값을 좀 더 유연하게 합쳐야 한다면 새로운 `merge` 메서드를 이용할 수 있다
두 맵에 `Cristina`가 다른 영화 값으로 존재한다면 `forEach`와 `merge` 메서드를 이용해 충돌을 해결할 수 있다
```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));  // 중복 키가 있다면 연결
```

`merge`를 이용해 초기화 검사를 구현할 수 있다
영화를 몇 회 시청했는지 기록하는 맵이 있을 때, 해당 값을 증가시키기 전에 관련 영화가 이미 맵에 존재하는지 확인이 필요

```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);  // 두 번째 인수 1L은 키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 값이 없거나 키에 널 값이면 연결
```

### 개선된 ConcurrentHashMap
`ConcurrentHashMap` 클래스는 동시성 친화적이며 최신 기술을 반영한 `HashMap` 버전이다
`ConcurrentHashMap`은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다
따라서 동기화된 `Hashtable` 버전에 비해 읽기 쓰기 연산 성능이 월등

#### 리듀스와 검색
- forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

`ConcurrentHashMap`의 상태를 잠그지 않고 연산을 수행한다는 점에서 이들 연산에 제공한 함수는 **계산이 진행되는 동안 바뀔 수 있는 객체, 값 순서 등에 의존하지 않아야 한다**

### 정리
- 자바 9는 적은 원소를 포함하며 바꿀 수 없는 `리스트`, `집합`, `맵`을 쉽게 만들 수 있도록 컬렉션 팩토리를 지원
- 컬렉션 팩토리가 반화난 객체는 만들어진 다음 바꿀 수 없다
- List 인터페이스는 `removeIf`, `replaceAll`, `sort` 세 가지 디폴트 메서드를 지원
- Set 인터페이스는 `removeIf` 디폴트 메서드를 지원
- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드를 지원
- `ConcurrentHashMap`은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공

## Chapter9 리팩터링, 테스팅, 디버깅
가독성과 유연성을 높이려면 기존 코드를 어떻게 리팩터링해야 하는지 설명한다
람다 표현식으로 `전략`, `템플릿 메서드`, `옵저버`, `의무 체인`, `팩토리` 등의 객체지향 디자인 패턴을 어떻게 간소화할 수 있는지도 살펴본다

- 익명 클래스를 람다 표현식으로 리팩터링
- 람다 표현식을 메서드 참조로 리팩터링
- 명령형 데이터 처리를 스트림으로 리팩터링

### 익명 클래스를 람다 표현식으로 리팩터링
```java
Runnable r1 = new Runnable() {    // 익명 클래스를 사용한 이전 코드
    public void runt() {
        System.out.println("Hello");
    }
}
```
```java
Runnable r2 = () -> System.out.println("Hello");
```
모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다
1. 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 가짐
    - 익명 클래스에서 this는 자신, 람다에서는 람다를 감싸는 클래스가 this 
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(섀도 변수)
    - 람다 표현식으로는 변수를 가릴 수 없다
  
```java
int a = 10;
Runnable r1 = new Runnable() {    // 익명 클래스를 사용한 이전 코드
    public void runt() {
        int a = 2;    // 정상 작동
        System.out.println("Hello");
    }
}
Runnable r2 = () -> {
        int a = 2;    // 컴파일 에러
        System.out.println("Hello");
```

3. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있음
```java
public static void doSomething(Runnable r) {r.run();}
public static void doSomething(Task r) {r.execute();}

doSomething(new Task() {
    public void execute() {
        System.out.println("Danger danger!!");
    }
}

// doSomething(() -> System.out.println("Danger danger!!"));    // Runnable, Task 모두 대상 형식이 될 수 있으므로 문제 발생
doSomething((Task)() -> System.out.println("Danger danger!!"));    // 명시적 형변환을 이용해 모호함 제거
```

### 람다 표현식을 메서드 참조로 리팩터링
람다 표현식을 별도의 메서드로 추출한 다음 메서드 참조로 리팩터링할 수 있다
메서드 참조로 리팩터링하면 코드가 간결하고 의도도 명확해진다
특히 메서드 참조와 조화를 이루도록 설계된 정적 헬퍼 메서드(`comparing`, `maxBy`, `sum`, `maximum`)를 활용하는 것도 좋다
혹은 내장 컬렉터를 이용하면 코드 자체로 문제를 명화하게 설명할 수 있음
```java
int totalCalories =
    menu.stream().map(Dish::getCalories)
                    .reduce(0, (c1, c2) -> c1 + c2);

int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

### 명령형 데이터 처리를 스트림으로 리팩터링
이론적으론 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림API로 바꿔야 한다고 한다
스트림 API는 데이터 처리 파이프라인의 의도를 명확ㅎ히 보여주며 `쇼트서킷`과 `게으름`이라는 강력한 최적화뿐 아니라 멀티코어 아키텍쳐를 활용할 수 있는 지름길을 제공한다

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu) {
        if(dish.getCalories() > 300) {
            dishNames.add(dish.getName());
        }
}
```
명령형 코드의 `break`, `continue`, `return` 등의 제어 흐름문을 모두 분석해서 같은 기능을 수행하는 스트림 연산으로 유추해야 하므로 명령형 코드를 스트림 API로 바꾸는 것은 쉬운일이 아니다
```java
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```

람다 표현식을 이용하려면 함수형 인터페이스가 필요하다 따라서 함수형 인터페이스를 코드에 추가해야 하는데, `조건부 연기 실행` 과 `실행 어라운드` 두 가지 자주 사용하는 패턴으로 리팩터링을 살펴본다

#### 조건부 연기 실행
코드 내부에 제어 흐름문이 복잡하게 얽힌 코드를 흔히 볼 수 있다

```java
if(logger.isLoggable(Log.FINER)) {
    logger.finer("Problem" + generateDiagnostic());
}
```
불필요한 if문을 제거할 수 있으며 logger의 상태를 노출할 필요도 없도록 `Supplier`를 인수로 갖는 오버로드된 log메서드를 제공
```java
public void log(Level level, Supplier<String> msgSupplier) {
    if(logger.isLoggable(level)) {
        log(level, msgSupplier.get());    // <- 람다 실행
    }    
```
#### 실행 어라운드
매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환할 수 있다

(1) 실행 어라운트 패턴의 구현
```java
public String processFile() throw IOException {
    try (
      BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
      return br.readLine(); // 실제 작업을 수행
    }
}
``` 

(2) 동작 파라미터화

위 (1)번 코드에서는 파일의 한줄씩 읽어들인다. 만약 파일을 한번에 두줄을 읽으려면 (실제 작업을 수정하려면) 기존의 설정/정리 과정은 재사용하고 실제 작업을 수행하는 한 줄의 코드만 수정하면 된다. 우리는 processFile의 동작을 파라미터화 시킬 수 있다.
```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

(3) 함수형 인터페이스
```java
@FunctionalInterface
public interface BufferedReaderProcessor {
  String process(BufferedReader b) throws IOExeption;
}

// 위 함수형 인터페이스를 processFile 메서드의 인수로 전달한다.
public String processFile(BufferedReaderProcessor p) throws IOException {
  ...
}
```

(4) 동작 실행
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며, 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다.
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
  try (
    BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
      return p.process(br);
    }
  )
}
```

(5) 람다 전달
이제 람다를 사용해서 다양한 동작을 processFile 메서드로 전달할 수 있다.
```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

- 람다로 `BufferedReader` 객체의 동작을 결정할 수 있는 것은 함수형 인터페이스 `BufferedReaderProcessor` 덕분이다

### 람다로 객체지향 디자인 패턴 리팩터링
디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는 검증된 청사진을 제공한다
디자인 패턴에 람다 표현식이 더해지면 색다른 기능을 발휘할 수 있다

#### 전략패턴

![스크린샷 2023-08-26 오후 9 16 50](https://github.com/dpwns523/modern-java-in-action/assets/84260096/97ff5ba0-4e8b-463c-9700-c1fb7185c75c)

- 알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
- 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현(ConcreteStrategyA, ConcreteStrategyB 같은 구체적인 구현 클래스)
- 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
Validator numericValidator =
    new Validator((String s) -> s.matches("[a-z]+"));    // 람다 직접 전달
boolean b1 = numericValidator.validate("aaaa");
Validator lowerValidator =
    new Validator((String s) -> s.matches("\\d+"));    // 람다 직접 전달
boolean b2 = lowerValidator.validate("bbbb");
```
- 람다 표현식을 이용하면 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있다 -> 코드 조각을 캡슐화한다

#### 템플릿 메서드
템플릿 메서드는 '이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는' 상황에 적합하다

다음은 온라인 뱅킹 애플리케이션의 동작을 정의하는 추상 클래스다
```java
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
    abstract void makeCustomerHaapy(Customer c);
}
```

람다나 메서드 참조로 알고리즘에 추가할 다양한 컴포넌트를 구현할 수 있다

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}
```
이제 `onlineBanking` 클래스를 상속받지 않고 직접 람다 표현식을 전달해서 다양한 동작을 추가할 수 있음
```java
new OnlineBankingLambda().processCustomer(1337, (Customer c) -> System.out.println("Hello " + c.getName()));
```

#### 옵저버 패턴
옵저버 패턴은 어떤 이벤트가 발생했을 때 한 객체(**주체**)가 다른 객체 리스트(**옵저버**)에 자동으로 알림을 보내야 하는 상황에서 사용된다
사용자가 버튼을 클릭하면 옵저버에 알림이 전달되고 정해진 동작이 수행

다양한 옵저버를 그룹화할 Observer 인터페이스가 필요한데, Observer 인터페이스는 새로운 호출이 있을 때 주체가 호출할 수 있도록 notify라고 하는 하나의 메서드를 제공한다

- 람다 표현식을 옵저버 디자인 패턴에 적용하여 다양한 `notify`를 구현할 수 있다

하지만, 무조건 람다 표현식을 사용해야 하진 않다
- 실행해야할 동작이 간단하면 람다 표현식으로 불필요한 코드를 제거하는 것이 바람직
- 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현방식을 고수하는 것이 바람직할 수 있음

#### 의무 체인
작업 처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다
- 한 객체가 어떤 작업을 처리한 다음 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식

작업 처리 객체를 andThen 메서드로 함수를 조합해서 체인을 만들 수 있음

#### 팩토리
인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용
다양한 상품을 만드는 `Factory` 클래스를 보자
```java
pbulc class ProductFactory {
    pbulic static Product createProduct(String name) {
        switch(name) {
            case "loan" : return new Loan();
            case "stock" : return new Stock();
            ...
        }
    }
}
```
람다 표현식을 사용하면 상품명을 생성자로 연결하는 Map을 만들어서 코드를 재구현할 수 있다
```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    ...
}
```

```java
pbulc class ProductFactory {
    pbulic static Product createProduct(String name) {
        Supplier<Product> p = map.get(name);
        if(p != null) return p.get();
        throw new IllegalArgumentException("No such product " + name);
    }
}
```
상품 생성자로 여러 인수를 전달하는 상황에서는 단순한 Supplier 함수형 인터페이스로는 이 문제를 해결할 수 없음

세 인수를 받는 상품의 생성자가 있다면 `TriFunction`이라는 특별한 함수형 인터페이스를 만들어야 한다

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

### 람다 테스팅
람다 표현식을 적용해 코드를 간결하게 리팩터링하였다 하지만 제대로 작동하는 코드를 구현했는지는 검증이 필요하다
**의도대로 동작하는지 확인할 수 있는 `단위 테스팅`을 진행해야 한다**
하지만 람다는 익명이므로 테스트 코드 이름을 호출할 수 없는데, 어떤 방식으로 테스트할 수 있는지 살펴본다

1. 보이는 람다 표현식의 동작 테스팅
   - 람다 표현식은 함수형 인터페이스의 인스턴스를 생성
       - 생성된 인스턴스의 동작으로 람다 표현식을 테스트
   - `Comparator` 객체 `compareByXAndThenY`에 다양한 인수로 compare 메서드를 호출하면서 예상대로 동작하는지 테스트
   ```java
   public void testComparingTwoPoints() throws Exception {
       Point p1 = new Point(10, 15);
       Point p2 = new Point(10, 20);
       int result = Point.compareByXAndThenY.compare(p1, p2);
       assertTrue(result < 0);
   }
   ``` 
2. 람다를 사용하는 메서드의 동작에 집중
    - 람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것
        - 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다
    - 람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다   
3. 복잡한 람다를 개별 메서드로 분할
    - 복잡한 람다 표현식을 메서드 참조로 바꾸는 것
    - 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있다
4.  고차원 함수 테스팅
    - 함수를 인수로 받거나 다른 함수를 반환하는 메서드 테스팅은 좀 더 어렵다
    - 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트 할 수 있다
    - 테스트해야할 메서드가 다른 함수를 반환한다면 -> 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트 할 수 있다

### 디버깅
코드를 테스트하면서 람다 표현식에 어떤 문제가 있음을 발견한다면 디버깅이 필요하다
문제가 발생한 코드를 디버깅할 때 두 가지 확인이 필요하다
- 스택 트레이스
- 로깅

#### 스택 트레이스
예외 발생으로 프로그램 실행이 갑자기 중단되었다면 스택 프레임에서 어떻게 멈췄는지 정보를 얻을 수 있다
람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성됨
- 스트림 파이프라인에서 에러가 발생하면 스트림 파이프라인 작업과 관련된 전체 메서드 호출 리스트가 출력된다
- 람다 표현식은 이름이 없으므로 컴파일러가 람다를 참조하는 이름을 만들어냄
    - 클래스에 여러 람다 표현식이 있을 때는 골치 아픈 일이 벌어진다
 
#### 정보 로깅
스트림의 파이프라인 연산을 디버깅한다면 `forEach`로 스트림 결과를 출력하거나 로깅할 수 있음
- 스트림 파이프라인에 적용된 각각의 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인할 수 있다

## Chapter10 람다를 이용한 도메인 전용 언어(DSL)
개발팀과 도메인 전문가가 공유하고 이해할 수 있는 코드는 생산성과 직결된다
-> 커뮤니케이션에서 버그와 오해를 방지하기 위해 이해하기 쉬운 코드는 특히 중요하다

도메인 전용 언어(domain-specific languages)로 애플리케이션의 비즈니스 로직을 표현함으로 이 문제를 해결할 수 있음
DSL이란 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어
모든 상황에서 적합한 것은 아니므로 장점과 비용을 모두 확인하고 DSL을 프로젝트에 추가하자

**장점**
- 간결함: API는 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피할 수 있고 코드를 간결하게 만듦
- 가독성: 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수 있음 -> 다양한 조직 구성원 간에 코드와 도메인 영역이 공유될 수 있음
- 유지보수: 잘 설계된 DSL로 구현한 코드는 쉽게 유지보수하고 바꿀 수 있음
- 높은 수준의 추상화: DSL은 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 관련되지 않은 세부 사항을 숨긴다
- 집중: 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어 -> 생산성이 좋아짐
- 관심사분리: 지정된 언어로 비즈니스 로직을 표현함으로 애플리케이션의 인프라 구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기 용이

**단점**
- DSL 설계의 어려움: 제한적인 언어에 도메인 지식을 담는 것이 쉬운 작업이 아님
- 개발 비용: 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모되는 작업 -> DSL 유지보수와 변경 부담
- 추가 우회 계층: DSL은 추가적인 계층으로 도메인 모델을 감싸며 이 때 계층을 쵣한 작게 만들어 성능 문제를 회피해야 함
- 새로 배워야 하는 언어: DSL을 추가하면서 팀이 배워야 하는 언어가 한 개 더 늘어남, 비즈니스 도메인을 다루는 개별DSL을 사용하는 상황이라면?
- 호스팅 언어 한계: 일부 자바 같은 범용 프로그래밍언어의 장황함과 엄격한 문법으로 DSL을 만들기가 힘듦 -> 자바의 람다는 이 문제를 해결할 강력한 도구

### 내부 DSL
자바로 구현한 DSL
역사적으로 자바는 다소 귀찮고, 유연성이 떨어지는 문법 때문에 읽기 쉽고, 간단하고, 표현력 있는 DSL을 만드는데 한계가 있었음
람다 표현식이 등장하면서 이 문제가 어느정도 해결
- 익명 내부 클래스 -> 람다 표현식
- 람다 표현식 -> 메서드 참조
**장점**
- 순수 자바로 DSL 을 구현하면 나머지 코드와 함께 컴파일 가능
    - 다른 언어의 컴파일러 또는 외부 DSL을 만드는 도구를 사용하지 않아도 됨
- 새로운 언어를 배우거나 복잡한 외부 도구를 배울 필요가 없음
- DSL 사용자는 기존의 자바 IDE로 자동 완성, 리팩터링 기능 그대로 사용 가능
- 추가 DSL을 쉽게 합칠 수 있음
    - 같은 자바 바이트코드를 사용하는 JVM 기반 프로그래밍 언어를 이용하므로 DSL 합침 문제를 해결하는 방법 -> 다중 DSL
 
### 다중 DSL
요즘 JVM에서 실행되는 언어는 100개가 넘으며 자바보다 젊으며 제약을 줄이고, 간편한 문법을 지향하도록 설계되었다
DSL은 기반 프로그래밍 언어의 영향을 받으므로 간결한 DSL을 만드는데 새로운 언어의 특성들이 아주 중요
- 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야 한다
- JVM 에서 실행되지만 자바와 호환성이 완벽하지 않은 경우가 많다

### 외부 DSL
프로젝트에 DSL을 추가하는 세 번째 옵션은 외부 DSL을 구현하는 것
자신만의 문법과 구문으로 새 언어를 설계해야 한다
- 외부 DSL을 구현하면 무한한 유연성이 있음
- 논리 정연한 프로그래밍 언어를 새로 개발한다는 것은 간단한 작업이 아님
    - 처음 설계한 목적을 벗어나는 경우도 많음
- 인프라 구조 코드와 외부DSL로 구현한 비즈니스 코드를 명확히 분리할 수 있음
    - **DSL과 호스트 언어 사이에 인공 계층이 생기므로 이는 양날의 검**
 
### 최신 자바 API의 작은 DSL
자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다

- 스트림 API는 컬렉션을 조작하는 DSL
- 데이터를 수집하는 DSL인 Collectors

### 스트림 API는 컬렉션을 조작하는 DSL

- Stream 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예
- 컬렉션의 항목을 필터, 정렬, 변환, 그룹화, 조작하는 작지한 강력한 DSL로 볼 수 있다
- Stream 인터페이스를 이용해 함수형으로 코드를 구현하면 쉽고 간결하게 코드를 구현할 수 있다

### 데이터를 수집하는 DSL인 Collectors
Stream 인터페이스를 데이터 리스트를 조작하는 DSL로 간주할 수 있다
마찬가지로 `Collector` 인터페이스는 데이터 수집을 수행하는 DSL로 간주할 수 있다

- 네이티브 자바 API의 내부 설계 결정 이유를 생각하면 가독성 있는 DSL을 구현하는 유용한 패턴과 기법을 배울 수 있다


### 자바로 DSL을 만드는 패턴과 기법

DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다
도메인 모델을 정의하면서 DSL을 만드는 패턴을 살펴본다

**도메인 모델은 세 가지로 구성**
1. 주어진 시장에 주식 가격을 모델링하는 순수 자바 빈즈
```java
public class Stock {
    private String symbol;
    private String market;
    /*
        getter와 setter
    */
    ...
}
```
2. 주어진 가격에서 주어진 양의 주식을 사거나 파는 거래
```java
public class Trade {
    public enum Type { BUY, SELL }
    private Type type;

    private Stock stock;
    private int quantity;
    private double price;
    /*
        getter와 setter
    */
    ...    
}
```
3. 고객이 요청한 한 개 이상의 거래의 주문
```java
public class Order {
    private String customer;
    private List<Trade> trades = new ArrayList<>();

    public void addTrade(Trade trade) {
        trades.add(trade);
    }
    /*
        getter와 setter
    */
    ...
}
```
도메인 객체의 API를 직접 이용해 주식 거래 주문을 만듦
```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.Buy);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

/*
    trade와 order, stock 값 set, add
*/
...
```
위의 코드는 상당히 장황한 편으로 비개발자인 도메인 전문가가 위 코드를 이해하고 검증하기를 기대할 수 없기 때문
-> 직접적이고, 직관적으로 도메인 모델을 반영할 수 있는 DSL이 필요

### 메서드 체인
DSL에서 가장 흔한 방식 중 하나
한 개의 메서드 호출 체인으로 거래 주문을 정의할 수 있음
```java
Order order = forCustomer("BigBank")
        .buy(80)
        .stock("IBM")
        .on("NYSE")
        .at(125.00)
        .sell(50)
        .stock("GOOGLE")
        .on("NASDAQ")
        .at(375.00)
        .end();
```
상당히 코드가 개선되었다
이 결과를 달성하려면 어떻게 DSL을 구현해야 할까? 
플루언트 API로 도메인 객체를 만드는 몇 개의 빌더를 구현해야 한다

**메서드 체인 DSL을 제공해주는 주문 빌더**
```java
public class MethodChainingOrderBuilder {

    public final Order order = new Order();    // <- 빌더로 감싼 주문
    
    private MethodChainingOrderBuilder(String customer) {
        order.setCustomer(customer);
    }

    public static MethodChainingOrderBuilder forCustomer(String customer) {
        return new MethodChainingOrderBuilder(customer);    // <- 고객의 주문을 만드는 정적 팩토리 메서드
    } 

    public TradeBuilder buy(int quantity) {
        return new TradeBuilder(this, Trade.Type.SELL, quantity);    // <- 주식을 사는 TradeBuilder 만들기
    }

    public TradeBuilder buy(int quantity) {
        return new TradeBuilder(this, Trade.Type.SELL, quantity);    // <- 주식을 파는 TradeBuilder 만들기
    }

    public MethodChainingOrderBuilder addTrade(Trade trade) {
        order.addTrade(trade);    // <- 주문에 주식 추가
        return this;
    }
    public Order end() {
        return order;    // <- 주문 만들기를 종료하고 반환
    }
}
```
TraderBuilder와 StockBuilder, 거래되는 주식의 단위 가격을 설정한 다음 원래 주문 빌더를 반환하는 TradeBuilderWithStock까지 빌더를 만들어야 한다
```java

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
```
- 사용자가 지정된 절차에 따라 플루언트 API의 메서드 호출하도록 강제할 수 있음
- 파라미터가 빌더 내부로 국한 됨 -> 정적 메서드 사용을 최소화하고 메서드 이름이 인수의 이름을 대신하여 가독성을 개선
- 플루언트 DSL에는 문법적 잡음이 최소화됨
단점
- 빌더를 구현해야 함
- 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요
- 도메인 객체의 중첩 구조와 일치하게 들여쓰기를 강제할 수 없음

### 중첩된 함수 이용
다른 함수 안에 함수를 이용해 도메인 모델을 만드는 방법

**중첩된 함수 DSL을 제공하는 주문 빌더**
```java
public class NestedFunctionOrderBuilder {

    public static Order order(String customer, Trade... trades) {
        Order order = new Order();
        order.setCustomer(customer);
        Stream.of(trades).forEach(order::addTrade);
        return order;
    }

    public static Trade buy(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.BUY);    // <- 주식 매수 거래 만들기
    }

    public static Trade sell(int quantity, Stock stock, double price) {
        return buildTrade(quantity, stock, price, Trade.Type.SELL);    // <- 주식 매도 거래 만들기
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
메서드 체인에 비해 함수의 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다는 것이 장점

단점
- 결과 DSL에 더 많은 괄호를 사용해야 함
- 인수 목록을 정적 메서드에 넘겨줘야 하는 제약
- 선택 사항 필드가 있으면 생략할 수 있으므로 여러 메서드 오버라이드를 구현해야 함

### 람다 표현식을 이용한 함수 시퀀싱
다음 DSL 패턴은 람다 표현식으로 정의한 함수 시퀀스를 사용
- 람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현해야 한다
- 빌더는 메서드 체인 패턴을 이용해 만들려는 객체의 중간 상태를 유지

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
        trade(consumer, Trade.Type.BUY);     // <- 주식 매수 주문을 만들도록 TradeBuilder 소비
    }

    public void sell(Consumer<TradeBuilder> consumer) {
        trade(consumer, Trade.Type.SELL);    // <- 주식 매도 주문을 만들도록 TradeBuilder 소비
    }

    private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
        TradeBuilder builder = new TradeBuilder();
        builder.trade.setType(type);
        consumer.accept(builder);
        order.addTrade(builder.trade);
    }
}

public class TradeBuilder {
    ...
}
```
주문 빌더의 buy(), sell() 메서드는 두 개의 Consumer<TradeBuilder> 람다 표현식을 받는다
이 패턴은 이전 두 가지 DSL 형식의 두 가지 장점을 더함
- 메서드 체인 패턴처럼 플루언트 방식으로 거래 주문을 정의할 수 있음
- 중첩 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지

단점
- 많은 설정 코드가 필요, DSL 자체가 자바 8 람다 표현식 문법에 의한 잡음의 영향을 받는다

### DSL에 메서드 참조 사용하기

```java
double value = new TaxCalculator().withTaxRegional()
                                  .withTaxSurcharge()
                                  .calculate(order);
```
**자바의 함수형 기능을 이용하면 더 간결하고 유연한 방식으로 같은 가독성을 달성할 수 있다**

### 실생활의 자바 8 DSL

![스크린샷 2023-08-27 오후 5 06 53](https://github.com/dpwns523/modern-java-in-action/assets/84260096/87e4fecb-2d1c-4f2f-a101-1a6a2cb8cf85)

세 가지 유명한 자바 라이브러리에 지금까지 살펴본 패턴이 얼마나 사용되고 있는지 살펴 본다

#### jOOQ -> SQL 매핑 도구
- jOOQ는 SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어
- jOOQ DSL 구현하는 데에는 메서드 체인 패턴을 사용
- SQL 문법을 흉내내려면 선택적 파라미터를 허용하고 정해진 순서대로 특정 메서드가 호출되어야 하기 때문에 메서드 체인 패턴의 특성이 필요
#### 큐컴버 -> 동작 주도 개발 프레임워크
- 동작 주도 개발(BDD)은 테스트 주도 개발의 확장으로 다양한 비즈니스 시나리오를 구조적으로 서술하는 도메인 전용 스크립팅 언어를 사용
- 큐컴버(cucumber)는 BDD 프레임워크로 명령문을 실행할 수 있는 테스트 케이스로 변환하며, 비즈니스 시나리오를 평문으로 구현할 수 있도록 도와줌
- 큐컴버는 세가지 구분되는 개념을 사용
- Given: 전제 조건 정의
- When: 시험하려는 도메인 객체의 실질 호출
- Then: 테스트 케이스의 결과를 확인
#### 스트링 통합 -> 엔터프라이즈 통합 패턴
- 스프링 통합은 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장
- 복잡한 통합 솔루션 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있도록 돕는 것이 스프링 통합의 핵심 목표
- 스프링 통합 DSL 에서도 메서드 체인 패턴이 가장 널리 사용되고 있음

### 정리
- DSL의 주요 기능은 개발자와 도메인 전문가 사이의 간격을 좁히는 것이다
- 애플리케이션의 비즈니스 로직을 구현하는 코드를 만든 사람이 프로그램이 사용될 비즈니스 필드의 전문 지식을 갖추긴 어렵다
- 개발자가 아닌 사람도 이해할 수 있는 언어로 이런 비즈니스 로직을 구현할 수 있다고 해서 도메인 전문가가 프로그래머가 될 수 있는 것은 아니지만 적어도 로직을 읽고 검증하는 역할은 할 수 있다 
- DSL은 내부DSL, 다중DSL, 외부DSL로 구분할 수 있으며 구분 기준은 호스팅 언어를 기반으로 한 정도이다
- JVM에서 이용할 수 있는 스칼라, 그루비 등 다른 언어로 다중 DSL을 개발할 수 있다
    - 장점: 자바보다 유연하며 간결
    - 단점: 자바와 통합할 때 빌드 과정이 복잡해지며 상호 호환성 문제 발생 가능
- 자바의 장황함과 문법적 엄격함 때문에 내부DSL 개발 언어로 적합하지 않다
- 자바 8의 람다 표현식과 메서드 참조 덕분에 상황이 많이 개선되었다

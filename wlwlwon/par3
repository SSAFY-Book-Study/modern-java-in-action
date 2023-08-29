## 8.1 컬렉션 팩토리

### Factory Method 이용

```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
/////////////////////////////////////////////////////////
List<String> friends
 = Arrays.asList("Raphael", "Olivia", "Thibaut")
```

Arrays.asList → 고정 크기의 리스트 생성, 요소 추가시 UnsupportedOperationException 예외 발생

### Set의 경우

```java
Set<String> friends 
 = new HashSet<>(Arrays.asList("Raphael", "Olivia", Thibaut"));
//////////////////////////////////////////////////////////////
Set<String> friends
 = Stream.of("Raphael", "Olivia", "Thibaut")
 .collect(Collectors.toSet());
```

## 8.1.1 리스트 팩토리

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
```

## 8.1.2. 집합 팩토리

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut"); System.out.
println(friends);
```

## 8.1.3 맵 팩토리

- 열개 이하의 키와 값 쌍을 가진 작은 맵을 만들때 유용
    
    ```java
    Map<String, Integer> ageOfFriends
     = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
    System.out.println(ageOfFriends);
    ```
    
- 그 이상의 맵 Map.Entry<K,V> 객체를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드 이용
- 

```java
import static java.util.Map.entry; 
Map<String, Integer> ageOfFriends
= Map.ofEntries(entry("Raphael", 30),
 entry("Olivia", 25),
 entry("Thibaut", 26));
System.out.println(ageOfFriends);
```

## 8.2 리스트와 집합 처리

- removeIf
- replaceAll
- sort

## 8.2.1 removeIf 메서드*************

```java
for (Transaction transaction : transactions) { 
 if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
 transactions.remove(transaction);
 }
}
```

- ConcurrentModificationException 발생
    - 이유 → for-each 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석

```java
for (Iterator<Transaction> iterator = transactions.iterator();
 iterator.hasNext(); ) {
 Transaction transaction = iterator.next();
 if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
 transactions.remove(transaction);
 }
}
```

- 두 개의 개별 객체가 컬렉션을 관리
- 반복자의 상태는 컬렉션의 상태와 서로 동기화 되지 않는다. Iterator 객체를 명시적으로 사용하고 그 객체의 remove() 메서드를 호출함으로 이 문제를 해결할 수 있다.

```java
transactions.removeIf(transaction -> 
 Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

## 8.2.2 replaceAll 메서드

- Stream 사용시
    
    ```java
    referenceCodes.stream() //[a12, C14, b13]
     .map(code -> Character.toUpperCase(code.charAt(0)) +
     code.substring(1))
     .collect(Collectors.toList())
     .forEach(System.out::println);
    ```
    
- 이 코드는 새로운 문자열 컬렉션을 만든다. 원하는것은 값을 바꾸는 일
- for-each 사용시
    
    ```java
    for (ListIterator<String> iterator = referenceCodes.listIterator();
     iterator.hasNext(); ) {
     String code = iterator.next();
     iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
    }
    ```
    
- replaceAll 사용시
    
    ```java
    referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.
    substring(1))
    ```
    

## 8.3.1 forEach 메서드

```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
 String friend = entry.getKey();
 Integer age = entry.getValue();
 System.out.println(friend + " is " + age + " years old");
}
//////////////////////////////////////////////////////
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
 age + " years old"));
```

## 8.3.2 정렬 메서드

```java
Map<String, String> favouriteMovies
 = Map.ofEntries(entry("Raphael", "Star Wars"),
 entry("Cristina", "Matrix"),
 entry("Olivia",
 "James Bond"));

favouriteMovies
 .entrySet()
 .stream()
 .sorted(Entry.comparingByKey())
 .forEachOrdered(System.out::println);
```

<aside>
💡 HashMap 성능

자바 8에서는 HashMap의 내부 구조를 바꿔 성능을 개선했다. 기존에 맵의 항목은 키로 생성한
해시코드로 접근할 수 있는 버켓에 저장했다. 많은 키가 같은 해시코드를 반환하는 상황이 되
면 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환해야 하므로 성능이 저하된다. 최근에는
버킷이 너무 커질 경우 이를 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해 동적으로 치
환해 충돌이 일어나는 요소 반환 성능을 개선했다. 하지만 키가 String, Number 클래스 같은
Comparable의 형태여야만 정렬된 트리가 지원된다.

</aside>

## 8.3.3 getOrDefault 메서드

```java
Map<String, String> favouriteMovies
 = Map.ofEntries(entry("Raphael", "Star Wars"),
 entry("Olivia", "James Bond"));
System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
```

## 8.3.4 계산 패턴

맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요한
때가 있다.

- computeIfAbsent : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
- compute : 제공된 키로 새 값을 계산하고 맵에 저장

## 8.3.5 삭제 패턴

```java
favouriteMovies.remove(key, value);
```

## 8.3.6 교체 패턴

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. 이 메서드는 이전에 살펴본 List의 replaceAll과 비슷한 동작을 수행한다.
- Replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

## 8.3.7  merge

```java
Map<String, String> family = Map.ofEntries(
 entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
 entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends); friends의 모든 항목을 everyone으로 복사
System.out.println(everyone);
```

중복된 키가 없다면 위코드는 잘 작동

forEach와 merge를 이용해 충돌을 해결할 수 있다.

```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) ->
 everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
System.out.println(everyone);
```

## 8.4 개선된 ConcurrentHashMap

- ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한
다. 따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다.

## 8.4.1 리듀스와 검색

- ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다는 점을 주목
- 병렬성 기준값threshold을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으
면 순차적으로 연산을 실행한다. 기준값을 1로 지정하면 공통 스레드 풀을 이용해 병렬성을 극
대화한다. Long.MAX_VALUE를 기준값으로 설정하면 한 개의 스레드로 연산을 실행한다.

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue =
 Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

## 8.4.2 계수

- ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. 기존의 size 메서드 대신 새 코드에서는 int를 반환하는 mappingCount 메서드를 사용하는 것이
좋다. 그래야 매핑의 개수가 **int의 범위를 넘어서는 이후의 상황을 대처할 수 있기 때문이다.**

## 

## 9.1 가독성과 유연성을 개선하는 리팩터링

- 람다, 메서드 참조, 스트림 등의 기능을 이용해서 더 가독성이 좋
고 유연한 코드로 리팩터링하는 방법

## 9.1.1 코드 가동성 개선

- 코드 가독성이란?
    - 어떤 코드를 다른 사람도 쉽게 이해할 수 있음
    - 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다.
    - 코드의 문서화를 잘하고, 표준 코딩 규칙을 준수하는 등의 노력을 기울여야한다.

## 9.1.2 익명 클래스를 람다 표현식으로 리팩터링 하기

- 람다 표현식을 이용해서 간결하고, 가독성이 좋은 코드를 구현할 수 있다.

```java
Runnable r1 = new Runnable() {
 public void run(){
 System.out.println("Hello");
 }
};
///////////////////////////////////////
Runnable r2 = () -> System.out.println("Hello");
```

- 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다
    - 첫째, 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다
        - 익명 클래스에서 this는 익명클래스 자신을 가리키지만 람다에서 this는 람다를 감싸는 클래스를 가리킨다.
    - 둘째, 익명클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다(섀도 변수shadow variable )

## 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기

```java

public class Dish{
 ...
 public CaloricLevel getCaloricLevel() {
 if (this.getCalories() <= 400) return CaloricLevel.DIET;
 else if (this.getCalories() <= 700) return CaloricLevel.NORMAL;
 else return CaloricLevel.FAT;
 }
}

////////////////////////////////////////

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
 menu.stream()
 .collect(
 groupingBy(dish -> {
 if (dish.getCalories() <= 400) return CaloricLevel.DIET;
 else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
 else return CaloricLevel.FAT;
}));

/////////////////////////////////////////

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
 menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```

## 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기

- 이론적으로는 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야 한다. 이
유가 뭘까?
    
    → 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다
    

## 9.1.5 코드 유연성 개선

- 즉, 다양한 람다를 전달해서 다양한 동작을 표현할 수 있다. 따라서 변화하는
요구사항에 대응할 수 있는 코드를 구현할 수 있다

## 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

- 다양한 패턴을 유형별로 정리한 것이 디자인 패턴이다.
- 디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공한다.

## 9.2.1 전략 패턴

- **전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기
법이다**
    
    ```java
    public interface ValidationStrategy {
     boolean execute(String s);
    }
    
    public class IsAllLowerCase implements ValidationStrategy {
     public boolean execute(String s) {
     return s.matches("[a-z]+");
     }
    }
    public class IsNumeric implements ValidationStrategy {
     public boolean execute(String s) {
     return s.matches("\\d+");
     }
    }
    
    public class Validator {
     private final ValidationStrategy strategy;
     public Validator(ValidationStrategy v) {
     this.strategy = v;
     }
     public boolean validate(String s) {
     return strategy.execute(s);
     }
    }
    Validator numericValidator = new Validator(new IsNumeric());
    boolean b1 = numericValidator.validate("aaaa"); false 반환
    Validator lowerCaseValidator = new Validator(new IsAllLowerCase ());
    boolean b2 = lowerCaseValidator.validate("bbbb"); true 반환
    ```
    
- 람다 표현식 사용
    
    ```java
    Validator numericValidator =
     new Validator((String s) -> s.matches("[a-z]+"));
    boolean b1 = numericValidator.validate("aaaa");
    Validator lowerCaseValidator =
     new Validator((String s) -> s.matches("\\d+"));
    boolean b2 = lowerCaseValidator.validate("bbbb");
    
    ```
    

## 9.2.2 템플릿 메서드

- 알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때
템플릿 메서드 디자인 패턴을 사용한다. 템플릿 메서드는 ‘이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는’ 상황에 적합하다
    
    ```java
    abstract class OnlineBanking {
     public void processCustomer(int id) {
     Customer c = Database.getCustomerWithId(id);
     makeCustomerHappy(c);
     }
     abstract void makeCustomerHappy(Customer c);
    }
    ```
    
- 람다 표현식 사용

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
 Customer c = Database.getCustomerWithId(id);
 makeCustomerHappy.accept(c);
}
new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
 System.out.println("Hello " + c.getName());
```

## 9.2.3 옵저버 패턴

- 어떤 이벤트가 발생했을 때 한 객체(주제subject라 불리는)가 다른 객체 리스트(옵저버observer라불리는)에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.
    
    ```java
    interface Observer {
     void notify(String tweet);
    }
    class NYTimes implements Observer {
     public void notify(String tweet) {
     if(tweet != null && tweet.contains("money")){
     System.out.println("Breaking news in NY! " + tweet);
    }
     }
    }
    class Guardian implements Observer {
     public void notify(String tweet) {
     if(tweet != null && tweet.contains("queen")){
     System.out.println("Yet more news from London... " + tweet);
     }
     }
    }
    class LeMonde implements Observer {
     public void notify(String tweet) {
     if(tweet != null && tweet.contains("wine")){
     System.out.println("Today cheese, wine and news! " + tweet);
     }
     }
    }
    
    interface Subject {
     void registerObserver(Observer o);
     void notifyObservers(String tweet);
    }
    
    class Feed implements Subject {
     private final List<Observer> observers = new ArrayList<>();
     public void registerObserver(Observer o) {
     this.observers.add(o);
     }
     public void notifyObservers(String tweet) {
     observers.forEach(o -> o.notify(tweet));
     }
    }
    Feed f = new Feed();
    f.registerObserver(new NYTimes());
    f.registerObserver(new Guardian());
    f.registerObserver(new LeMonde());
    f.notifyObservers("The queen said her favourite book is Modern Java in Action!")
    ```
    
- 람다 표현식 사용하기
    
    ```java
    f.registerObserver((String tweet) -> {
     if(tweet != null && tweet.contains(“money”)){
     System.out.println(“Breaking news in NY! “ + tweet);
     }
    });
    f.registerObserver((String tweet) -> {
     if(tweet != null && tweet.contains(“queen”)){
     System.out.println(“Yet more news from London... “ + tweet);
     }
    });
    ```
    

## 9.2.4 의무 체인

- 작업 처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다

```java
public abstract class ProcessingObject<T> {
 protected ProcessingObject<T> successor;
 public void setSuccessor(ProcessingObject<T> successor){
 this.successor = successor;
 }
 public T handle(T input) {
 T r = handleWork(input);
 if(successor != null){
 return successor.handle(r);
 }
 return r;
 }
 abstract protected T handleWork(T input);
}
```

9.2.5 팩토리

- 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

```java
public class ProductFactory {
 public static Product createProduct(String name) {
 switch(name){
 case “loan”: return new Loan();
 case “stock”: return new Stock();
 case “bond”: return new Bond();
 default: throw new RuntimeException(“No such product “ + name);
 }
 }
}
```

- 람다 표현식

```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
 map.put("loan", Loan::new);
 map.put("stock", Stock::new);
 map.put("bond", Bond::new);
}
```

## 9.3 람다 테스팅

- 일반적으로 좋은 소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 단위 테스팅unit testing을 진행한다.

## 9.3.1 보이는 람다 표현식의 동작 테스팅

- 람다 표현식은 함수형 인터페이스의 인스턴스를 생성한다는 사실을 기억

## 9.3.2 람다를 사용하는 메서드의 동작에 집중하라

- 람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는것이다

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
 return points.stream()
 .map(p -> new Point(p.getX() + x, p.getY()))
 .collect(toList());
}
```

```java
@Test
public void testMoveAllPointsRightBy() throws Exception {
 List<Point> points =
 Arrays.asList(new Point(5, 5), new Point(10, 5));
 List<Point> expectedPoints =
 Arrays.asList(new Point(15, 5), new Point(20, 5));
 List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
 assertEquals(expectedPoints, newPoints);
}
```

## 9.4 디버깅

문제가 발생한 코드를 디버깅할 때 개발자는 다음 두 가지를 가장 먼저 확인해야 한다.
● 스택 트레이스
● 로깅
하지만 람다 표현식과 스트림은 기존의 디버깅 기법을 무력화한다. 8.4절에서는 람다 표현식
과 스트림 디버깅 방법을 살펴본다

## 9.4.1 스택 트레이스 확인

예를 들어 예외 발생으로 프로그램 실행이 갑자기 중단되었다면 먼저 어디에서 멈췄고 어떻게
멈추게 되었는지 살펴봐야 한다. 바로 **스택 프레임stack frame**에서 이 정보를 얻을 수 있다. 프로그램이 메서드를 호출할 때마다 프로그램에서의 호출 위치, 호출할 때의 인수값, 호출된 메서드
의 지역 변수 등을 포함한 호출 정보가 생성되며 이들 정보는 스택 프레임에 저장된다.

```java
import java.util.*;
public class Debugging{
 public static void main(String[] args) {
 List<Point> points = Arrays.asList(new Point(12, 2), null);
 points.stream().map(p -> p.getX()).forEach(System.out::println);
 }
}

//에러
Exception in thread "main" java.lang.NullPointerException
 at Debugging.lambda$main$0(Debugging.java:6) 여기 $0은 무슨 의미일까?
 at Debugging$$Lambda$5/284720968.apply(Unknown Source)
 at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline
 .java:193)
 at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators
 .java:948)
...
```

이와 같은 이상한 문자는 람다 표현식 내부에서 에러가 발생했음을 가리킨다. 람다 표현식은
이름이 없으므로 컴파일러가 람다를 참조하는 이름을 만들어낸 것이다. lambda$main$0는 다소
생소한 이름이다. 클래스에 여러 람다 표현식이 있을 때는 꽤 골치 아픈 일이 벌어진다.

## 9.2.4 정보 로깅

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);
numbers.stream()
 .map(x -> x + 17)
 .filter(x -> x % 2 == 0)
 .limit(3)
 .forEach(System.out::println);
```

## 10 람다를 이용한 도메인 전용 언어

도메인 전용 언어(DSL)로 애플리케이션의 비즈니스 로직을 표현함으로 이 문제를 해결할 수
있다. DSL은 작은, 범용이 아니라 특정 도메인을 대상으로 만들어진 특수 프로그래밍 언어다.
DSL은 도메인의 많은 특성 용어를 사용한다. 

## 10.1 도메인 전용 언어

- 의사 소통의 왕
    - 우리의 코드의 의도가 명확히 전달되어야 하며 프로그래머가 아닌 사람
    도 이해할 수 있어야 한다. 이런 방식으로 코드가 비즈니스 요구사항에 부합하는지 확인
    할 수 있다.
- 한 번 코드를 구현하지만 여러 번 읽는다
    - 가독성은 유지보수의 핵심이다. 즉 항상 우리의 동료가 쉽게 이해할 수 있도록 코드를 구현해야 한

## 10.1.1 DSL의 장점과 단점

장점

- 간결함 : API는 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피할 수 있고 코드를 간
결하게 만들 수 있다.
- 가독성 : 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수
있다. 결과적으로 다양한 조직 구성원 간에 코드와 도메인 영역이 공유될 수 있다.
- 유지보수 : 잘 설계된 DSL로 구현한 코드는 쉽게 유지보수하고 바꿀 수 있다. 유지보수는
비즈니스 관련 코드 즉 가장 빈번히 바뀌는 애플리케이션 부분에 특히 중요하다.
- 높은 수준의 추상화 : DSL은 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제
와 직접적으로 관련되지 않은 세부 사항을 숨긴다.
- 집중 : 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어이므로 프로그래머가 특
정 코드에 집중할 수 있다. 결과적으로 생산성이 좋아진다.
- 관심사분리Separation of concerns : 지정된 언어로 비즈니스 로직을 표현함으로 애플리케이션의 인프라구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기가 용이하다. 결과적으로 유지보수가 쉬운 코드를 구현한다

단점

- DSL 설계의 어려움 : 간결하게 제한적인 언어에 도메인 지식을 담는 것이 쉬운 작업은
아니다.
- 개발 비용 : 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모되
는 작업이다. 또한 DSL 유지보수와 변경은 프로젝트에 부담을 주는 요소다.
- 추가 우회 계층 : DSL은 추가적인 계층으로 도메인 모델을 감싸며 이 때 계층을 최대한
작게 만들어 성능 문제를 회피한다.
- 새로 배워야 하는 언어 : 요즘에는 한 프로젝트에도 여러가지 언어를 사용하는 추세다.
하지만 DSL을 프로젝트에 추가하면서 팀이 배워야 하는 언어가 한 개 더 늘어난다는 부
담이 있다. 여러 비즈니스 도메인을 다루는 개별 DSL을 사용하는 상황이라면 이들을 유기적으로 동작하도록 합치는 일은 쉬운 일이 아니다. 개별 DSL이 독립적으로 진화할 수
있기 때문이다.
- 호스팅 언어 한계 : 일부 자바 같은 범용 프로그래밍 언어는 장황하고 엄격한 문법을 가
졌다. 이런 언어로는 사용자 친화적 DSL을 만들기가 힘들다. 사실 장황한 프로그래밍 언
어를 기반으로 만든 DSL은 성가신 문법의 제약을 받고 읽기가 어려워진다. 자바 8의 람다
표현식은 이 문제를 해결할 강력한 새 도구다

## 10.1.2 JVM에서 이용할 수 있는 다른 DSL 해결책

- 내부 DSL
    - 내부 DSL이란 자바로 구현한 DSL을 의미한다.
    - 기존 자바 언어를 이용하면 외부 DSL에 비해 새로운 패턴과 기술을 배워 DSL을 구현하는 노력이 현저하게 줄어든다.
    - 순수 자바로 DSL을 구현하면 나머지 코드와 함께 DSL을 컴파일할 수 있다. 따라서 다른
    언어의 컴파일러를 이용하거나 외부 DSL을 만드는 도구를 사용할 필요가 없으므로 추가
    로 비용이 들지 않는다.
    - 여러분의 개발 팀이 새로운 언어를 배우거나 또는 익숙하지 않고 복잡한 외부 도구를 배
    울 필요가 없다.
    - DSL 사용자는 기존의 자바 IDE를 이용해 자동 완성, 자동 리팩터링 같은 기능을 그대로
    즐길 수 있다. 최신 IDE는 다른 유명한 JVM 언어도 지원하지만 자바 만큼의 기능을 지원
    하진 못한다.
    - 한 개의 언어로 한 개의 도메인 또는 여러 도메인을 대응하지 못해 추가로 DSL을 개발해
    야 하는 상황에서 자바를 이용한다면 추가 DSL을 쉽게 합칠 수 있다.
- 다중 DSL
    - 새로운 프로그래밍 언어를 배우거나 또는 팀의 누군가가 이미 해당 기술을 가지고 있어
    야 한다. 멋진 DSL을 만들려면 이미 기존 언어의 고급 기능을 사용할 수 있는 충분한 지
    식이 필요하기 때문이다.
    - 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선
    해야 한다.
    - 마지막으로 JVM에서 실행되는 거의 모든 언어가 자바와 백 퍼센트 호환을 주장하고 있
    지만 자바와 호환성이 완벽하지 않을 때가 많다. 이런 호환성 때문에 성능이 손실될 때도
    있다. 예를 들어 스칼라와 자바 컬렉션은 서로 호환되지 않으므로 상호 컬렉션을 전달하
    려면 기존 컬렉션을 대상 언어의 API에 맞게 변환해야 한다.

## 10.2 최신 자바 API의 작은 DSL

- 자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다.

```java
Collections.sort(persons, new Comparator<Person>() {
 public int compare(Person p1, Person p2) {
 return p1.getAge() - p2.getAge();
 }
});

//람다 표현식
Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge());

//Comparator.comparing
Collections.sort(persons, comparing(p -> p.getAge()));

//메서드 참조
Collections.sort(persons, comparing(Person::getAge));
```

## 10.3 자바로 DSL을 만드는 패턴과 기법

DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높는 API를 제공한다.

## 10.3.1 메서드 체인

```java
Order order = forCustomer( "BigBank" )
 .buy( 80 )
 .stock( "IBM" )
 .on( "NYSE" )
 .at( 125.00 )
 .sell( 50 )
 .stock( "GOOGLE" )
 .on( "NASDAQ" )
 .at( 375.00 )
 .end();
```

## 10.3.2 중첩된 함수 이용

중첩된 함수 DSL 패턴은 이름에서 알 수 있듯이 다른 함수 안에 함수를 이용해 도메인 모델을
만든다.

```java
Order order = order("BigBank",
 buy(80,
 stock("IBM", on("NYSE")), at(125.00)),
 sell(50,
 stock("GOOGLE", on("NASDAQ")), at(375.00))
 );
```

## 10.3.3 람다 표현식을 이용한 함수 시퀀싱

```java
Order order = order( o -> {
 o.forCustomer( "BigBank" );
 o.buy( t -> {
 t.quantity( 80 );
 t.price( 125.00 );
 t.stock( s -> {
 s.symbol( "IBM" );
 s.market( "NYSE" );
 });
 });
 o.sell( t -> {
 t.quantity( 50 );
 t.price( 375.00 );
 t.stock( s -> {
 s.symbol( "GOOGLE" );
 s.market( "NASDAQ" );
 });
 });
});
```

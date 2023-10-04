# 컬렉션 API 개선
## 1. 컬렉션 팩토리 
### 팩토리 메서드 사용
#### 리스트 팩토리 : List.of
- `Arrays.asList(1, 2, 3)` 와 같은 메서드를 사용 할 수도 있다.
- 위와 같은 코드로 생성된 `List<T>` 는 불변의 크기를 가진다.
	- update는 가능
	- add는 불가능
 - `List.of("Raphael", "Olivia", "Thibaut")` 팩토리 메소드를 사용한다.
#### 집합 팩토리 : Set.of
- asSet() 이라는 메서드는 존재하지 않는다.
- `Set.of("Raphael", "Olivia", "Thibaut")` 팩토리 메소드를 사용한다.
- `Set.of("Raphael", "Olivia", "Olivia")` 같은 요소를 중복하면 `IllegalArgumentException` 발생
#### Map으로의 변환
- `Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26)` 팩토리 메소드를 사용한다.
	- 키와 값을 번갈아 입력으로 준다.
```java
Map.ofEntries(entry("Raphael", 30),
            entry("Olivia", 25),
            entry("Thibaut", 26));
```
- 위와 같이 key, value를 묶은 `entry`를 주는 방식으로도 만들 수 있다.

## 2. 리스트와 집합 처리
- `removeIf( predicate )` :  프레디케이트가 true인 경우 요소 삭제
- `replaceAll( function )` : 모든 요소들에 대해 function을 실행한 것을 대신하여 저장

## 3. 맵 처리
- **forEach 메서드 : BiConsumer(키와 값을 인수로 받음)를 인수로 받는 for Each메서드 지원.**
	- `ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));`
- **정렬 메서드 : 키 또는 값으로 정렬 가능**
	- comparingByKey() 또는 comparingByValue() 가능
	- ```java
	    favouriteMovies
       .entrySet()
       .stream()
       .sorted(Entry.comparingByKey())
       .forEachOrdered(System.out::println);
        ```

- getOrDefault 메서드 : 첫 번째 인수로 키, 두 번째 인수로 기본값을 받는다
- **계산 메서드 : compute**
	- computeIfAbsent : 제공된 키에 해당하는 값이 없으면 (값이 없거나 null), 키를 이용해 새 값을 계산하고 맵에 추가
	- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
	- compute : 제공된 키로 새 값을 계산하고 맵에 저장
		- 키가 존재하지 않을 때에만 새 값을 추가하는 방식으로 정보를 캐시할 수 있다.
- **삭제 메서드 : remove**
	- 맵에서의 원소 삭제를 `avouriteMovies.remove(key, value)`와 같이 간단하게 할 수 있다.
		- key가 존재하고 그 값이 value와 같다면 삭제해준다.
- **교체 패턴 : replace**
	- replaceAll : BiFunction을 적용한 결과로 각 항목의 값 교체
	- Replace : 키가 존재하면 맵의 값을 바꾼다.
- **합침 : putAll, forEach( merge )**
	- 중복된 키에 대한 걱정이 없을 때, Map.putAll(Map)을 사용할 수 있다.
	- 중복된 키를 만났을 때 어떻게 처리할 지 정해주는 BiFunction을 인수로 주어 merge를 할 수도 있다.
```java
friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2)  -> movie1 + " & " movie2));
// 영화를 몇 회 시청했는지 기록하는 방법도 있다.
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
// 1L 초기값, count+1L이 다음 값이 된다.
```

## 4. 개선된 ConcurrentHashMap
- 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.
- 읽기 쓰기 연산 성능이 월등하다.
- **리듀스와 검색**
	- ConCurrentHashMap에서 지원하는3 가지 새로운 연산
	- forEach : 각 키, 값 쌍에 주어진 액션을 실행
	- reduce : 모든 키, 값 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
	- search : null이 아닌 값을 반환할 때까지 각 키, 값쌍에 함수를 적용
- **계수**
	- mappingCount : 맵의 매핑 개수 반환
-  **집합뷰**
	- keySet이라는 메서드 제공.
	- 맵을 바꾸면 집합도 바뀌고 집합을 바꾸면 맵도 영향을 받는다!

# 리팩터링, 테스팅, 디버깅
## 1. 가독성과 유연성을 개선하는 리팩터링
- 가독성이 좋다 = '어떤 코드를 다른 사람도 쉽게 이해할 수 있음'
	- **익명 클래스를 람다 표현식으로 리팩터링하기**
		- `Runnable r = () -> System.out.println("Hello")`
		- 위의 코드에서 Runnable의 run을 구현하기 위해 익명 클래스를 만들 필요가 없다.
			- 람다이므로 로컬 변수를 사용할 때 제약이 있을 수 있다. 상황에 맞는 사용 필요
	- **람다 표현식을 메서드 참조로 리팩터링하기**
		- `inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()))`
		- `inventory.sort(comparing(Apple::getWeight))`
		- 아래의 코드가 문제 자체를 설명하므로 더 의도가 명확해진다.
	- **명령형 데이터 처리를 스트림으로 리팩터링하기**
		- 문제를 더 직접적으로 기술할 수 있고, 쉽게 병렬화 할 수 있다.
		- ```java
		   menu.parallelStream()
       .filter(d -> d.getCalories() > 300)
       .map(Dish::getName)
       .collect(toList());
			```
#### 코드 유연성 개선
- 람다 표현식을 이용하기 위해선 함수형 인터페이스 적용 필요
	- 조건부 연기 실행과 실행 어라운드 패턴으로 람다 표현식 리팩터링
##### 조건부 연기 실행
```java
if (logger.isLoggable(Log.FINER)) {
	 logger.finer("Problem: " + generateDiagnostic());
} // 위와 같이 외부에서 상태를 확인 해 주는 것보다
logger.log(Level.FINER, "Problem: " + generateDiagnostic());
// 내부적으로 객체의 상태를 확인하는 코드를 통해
// 가독성과 캡슐화 강화 가능
```

##### 실행 어라운드
- 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 이를 람다로 변환할 수 있다.
```java
public static String processFile(BufferedReaderProcessor p) throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader(“ModernJavaInAction/chap9/data.txt”))) {
	return p.process(br); 
	}
}
// 위와 같은 함수를 만들어 두고 BufferedReaderProcessor 자리에 람다를 전달하여 비슷한 다른 동작을 실행할 수 있다.
```

## 2. 람다로 객체지향 디자인 패턴 리팩터링하기

1. 전략 : 런타임에 적절한 알고리즘을 선택하는 기법

```java
public interface ValidationStrategy {
	boolean execute(String s);
} // 이는 함수형 인터페이스이며 Predicate<String>과 같은 함수 디스크럽터를 갖고 있다.

public class Validator {
	private final ValidationStrategy strategy;
	public Validator(ValidationStrategy v) {
		this.strategy = v;
	}
	public boolean validate(String s) {
		return strategy.execute(s);
	}
} // 위와 같은 클래스를 구현하여 람다를 받을 수 있도록 만들 수 있다.

Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa");
// 위와 같이 람다를 직접 전달하여 새로운 전략 가진 validator를 만들 수 있다.
```

2. 템플릿 메서드
	- 이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는 상황에 적합하다.
```java
abstract class OnlineBanking {  
	public void processCustomer(int id) {
		Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
	abstract void makeCustomerHappy(Customer c);
}
// 위의 추상 클래스를 상속받아 makeCustomerHappy 함수를 구현할 수 있다.
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
	Customer c = Database.getCustomerWithId(id);
	makeCustomerHappy.accept(c);
}
new OnlineBankingLambda().processCustomer(1337, (Customer c) -> System.out.println("Hello " + c.getName()));
// 클래스를 상속받지 않고도 직접 람다 표현식을 전달해서 다양한 동작을 추가할 수 있다.
```

3. 옵저버
	- 어떤 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야 하는 상황에서 사용
```java
interface Observer {  
	void notify(String tweet);
}

interface Subject {  
	void registerObserver(Observer o); void notifyObservers(String tweet);
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

// main에서
Feed f = new Feed();
f.registerObserver((String tweet) -> {
	if(tweet != null && tweet.contains("money")){
		System.out.println("Breaking news in NY! " + tweet);
	}
});
f.registerObserver((String tweet) -> {
	if(tweet != null && tweet.contains("queen")){
		System.out.println("Breaking news from London... " + tweet);
	}
});
// 위와 같이 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 직접 전달하여 실행할 동작 지정 가능
```

4. 의무 체인
- 작업 처리 객체의 체인을 만들 때 사용한다.
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
// 작업 처리 객체를 Function<String, String>, 더 정확하게는UnaryOperator<String> 형식의 인스턴스로 표현할 수 있다.
// andThen 메서드로 이들 함수를 조합해서 체인을 만들 수 있다!!
UnaryOperator<String> headerProcessing =
	(String text) -> "From Raoul, Mario and Alan: " + text;
UnaryOperator<String> spellCheckerProcessing =
	(String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline =
	headerProcessing.andThen(spellCheckerProcessing);
	// 두 함수를 조합한다.
String result = pipeline.apply("Aren't labdas really sexy?!!");
// 결과 : From Raoul, Mario and Alan: Aren't lambdas really sexy?!!
```

5. 팩토리
- 다양한 상품을 만드는 상황
```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
	map.put("bond", Bond::new);
}
public static Product createProduct(String name){
	Supplier<Product> p = map.get(name);  
	if(p != null) return p.get();  
	throw new IllegalArgumentException("No such product " + name);
}
// 상품 생성자로 여러 인수를 전달하지 않는다면
// 위와 같이 자바 8의 기능들로 깔끔하게 정리할 수 있다.
```

## 3. 람다 테스팅
- 람다를 실행한 결과를 `assertTrue()` 등의 테스팅 함수를 이용해 검사할 수 있다.
- equals 메서드를 적절하게 구현해야 제대로 된 검사를 할 수 있다.

## 4. 디버깅
- 디버깅 할 때 개발자가 가장 먼저 확인해야 할 두가지
1. 스택 트레이스
	- 람다 표현식은 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성된다.
	- 예) `lambda$main$0`
	- 메서드 참조는 클래스와 같은 곳에 선언되어 있는 메서드를 참조할 때는 이름이 스택트레이스에 나타난다.
2. 정보 로깅
	- peek이라는 스트림 연산은 스트림의 요소들을 확인하여 **각 요소를 소비한 것처럼 동작을 실행하지만, 실제로 스트림의 요소를 소비하지는 않는다**.
```java
numbers.stream()
		.peek(x -> System.out.println("from stream: " + x))
		.map(x -> x + 17)
		.peek(x -> System.out.println("after map: " + x))
		.filter(x -> x % 2 == 0)
		.peek(x -> System.out.println("after map: " + x))
		.limit(3)
		.peek(x -> System.out.println("after map: " + x))
		.collect(toList());
// 스트림의 요소가 2, 3, 4, 5 일 때 결과
//   from stream: 2
//   after map: 19
//   from stream: 3
//   after map: 20
//   after filter: 20
//   after limit: 20
//   from stream: 4
//   after map: 21
//   from stream: 5
//   after map: 22
//   after filter: 22
//   after limit: 22
```


# 람다를 이용한 도메인 전용 언어
- 스트림 API의 특성인 메서드 체인을 보통 자바의 루프의 복잡함 제어와 비교하여 **플루언트 스타일**이라고 부른다.
## 1. 도메인 전용 언어 : DSL
- 프로그래머가 아닌 사람도 이해할 수 있어야 한다.
- 한번 코드를 구현하지만 여러 번 읽는다 -> 쉽게 이해할 수 있도록 코드를 구현해야 한다.
- **장점**
	- 간결함
	- 가독성
	- 유지보수에 좋다
	- 도메인 문제와 직접적으로 관련되지 않은 세부사항을 숨긴다.
	- 프로그래머가 특정 코드에 집중할 수 있다.
	- 인프라 구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기 용이하다. -> 유지 보수가 쉬운 코드를 구현한다.
- **단점**
	- DSL 설계의 어려움
	- 개발 비용. 초기 프로젝트에 DSL을 추가하는 작업은 많은 비용과 시간이 소모된다.
	- 새로 배워야 하는 언어
	- 호스팅 언어의 한계를 그대로 따른다.
		- (java와 같이 장황하고 엄격한 언어로는 사용자 친화적 DSL을 만들기가 힘들다.)
### 내부 DSL : 자바로 구성
- 람다의 등장으로 인해 **신호 대비 잡음 비율**을 줄일 수 있었다.
```java
List<String> numbers = Arrays.asList("one", "two", "three"); numbers.forEach( new Consumer<String>() {
	@Override  
	public void accept( String s ) {
        System.out.println(s);
	}
});
// 위의 코드를
numbers.forEach(s -> System.out.println(s));
// 위와 같이 변경 가능
numbers.forEach(System.out::println);
// 메소드 참조로 더 간단하게도 만들 수 있다.
```
- 새로운 언어를 배워 DSL을 구현하는 노력이 필요없다.
- 코드와 함께 DSL을 컴파일 할 수 있다. -> 비용 감소

### 다중 DSL : jvm 기반 프로그래밍 언어
- 자바보다 젊은 언어들을 이용하여 제약을 줄이고, 간편한 문법을 지향한다.
```
3 times {
    println("Hello World")
}
// 위와 같은 DSL을 통해 Hello World를 3번 출력 할 수 있다.
// DSL 친화적
```
- 새로운 언어를 배우거나 다른 누군가 해당 언어에 대한 기술을 가지고 있어야한다.
- 여러 컴파일러로 소스를 빌드하도록 빌드 과정을 개선해야한다.
- 100% jvm과 호환되지 않는 경우가 많다. -> 성능 손실 발생 가능

### 외부 DSL : 자신만의 문법과 구문으로 새 언어 설계
- 가장 큰 장점으로 **무한한 유연성**을 통해 **우리에게 필요한 특성을 완벽하게 제공하는 언어를 설계할 수 있다.**

## 2. 최신 자바 API의 작은 DSL
- 람다가 어떻게 네이티브 자바 API의 재사용성과 메서드 결합도를 높였는가?
```java
Collections.sort(persons, new Comparator<Person>() {
	public int compare(Person p1, Person p2) {
        return p1.getAge() - p2.getAge();
    }
});
// 위와 같은 코드를
Collections.sort(persons, comparing(p -> p.getAge()));
// 위와 같이 변경할 수 있도록 했다.
Collections.sort(persons, comparing(Person::getAge));
// 메서드 참조로 대신해 코드를 개선할 수도 있다.
persons.sort(comparing(Person::getAge)
    .thenComparing(Person::getName));
// list의 sort 메서드와 comparing, thenComparing, 그리고 메서드참조까지 이용한 예제
```

- **스트림 API** : 컬렉션을 조작하는 DSL
	- 스트림 API의 플루언트 형식은 잘 설계된 DSL의 또 다른 특징이다
		- 모든 중간 연산은 게으르다
		- 다른 연산으로 파이프라인 될 수 있다
		- 최종 연산은 적극적이면 전체 파이프라인이 계산을 일으킨다.
- **Collectors** : 데이터 수집을 수행하는 DSL
	- groupingBy 팩터리 메서드에 작업을 위임하는 GroupingBuilder를 만들 수 있다.
	- 하지만 중첩된 그룹화 수준에 반대로 그룹화 함수를 구현해야 한다.
		- 코드가 직관적이지 못하다.
```java
public class Stock {
	private String symbol;
	private String market;
	// getter, setter ... 
}

public class Trade {
	public enum Type {BUY, SELL}
	private Type type;
	private Stock stock;
	private int quantity;
	private double price;
	// getter, setter ...
}

public class Order {
	private String customer;
	private List<Trade> trades = new ArrayList<>();
	public void addTrade(Trade trade) {
		trades.add(trade);
	}
	// customer getter, setter
	public double getValue() {
		return trades.stream().mapToDouble(Trade::getValue).sum();
	}
}
```
- 위와 같은 주식 거래 시스템을 만들었을 때 이를 get, set등을 사용하여 조작하는 것은 상당히 장황한 코드를 만든다.
- DSL로 더 쉽게 접근하는 법들을 정리
1. 메서드 체인
```java
Order order = forCustomer( "BigBank" ) .buy( 80 )
           .stock( "IBM" )
           .on( "NYSE" )
           .at( 125.00 )
           .sell( 50 )
           .stock( "GOOGLE" )
           .on( "NASDAQ" )
           .at( 375.00 )
           .end();
// 위와 같이 메서드 체인을 통해 거래 주문을 정의할 수 있게 만드는 방법
// MethodChainingOrderBuilder 클래스를 만든다.
// MethodChainingOrderBuilder를 반환하는 forCustomer(string customer) 함수
// TradeBuilder를 반환하는 buy(int quantity)를 만들어 주식을 살 수 있게한다.
// TradeBuilder를 반환하는 sell(int quantity)를 만들어 주식을 팔 수 있게한다.
// MethodChainingOrderBuilder를 반환하는 addTrade(Trade trade)를 만들어 주문에 주식을 추가할 수 있게 한다.
// 주문 만들기를 종료하고 반환할 Order end() 함수를 만들고 중간 연산들로 만든 order를 반환한다.
```
- 빌더를 구현해야 한다는 것이 메서드 체인의 단점이다.

2. 중첩된 함수 이용
```java
Order order = order("BigBank", 
				buy(80,
		            stock("IBM", on("NYSE")), at(125.00)),
				sell(50,
                    stock("GOOGLE", on("NASDAQ")), at(375.00))
                );
// 위와 같이 중첩된 함수를 통해 주문을 정의할 수 있게 만드는 방법
// NestedFunctionOrderBuilder를 만든다.
// Order를 반환하는 order를 만들어 해당 고객의 이름String과 주문에 거래를 추가하는 매개변수를 가변 매개변수(Trade... trades)로 만든다.
// Trade를 반환하는 buy 함수를 만든다. 매수 거래를 만든다.
// Trade를 반환하는 sell 함수를 만든다. 매도 거래를 만든다.
// 거래된 주식의 단가를 정의하는 더미 메서드 at을 만든다.
// 거래된 주식Stock을 반환하는 stock메서드를 만든다.
// 주식이 거래된 시장을 정의하는 더미 메서드 on을 만든다.
```
- 메서드 체인에 비해 중첩 방식이 도메인 객체 계층 구조에 그대로 반영된다는 장점이 있다.
- DSL에 더 많은 괄호를 사용한다는 단점이 있다.

3. 람다 표현식을 이용한 함수 시퀀싱
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
// 위와 같이 람다를 이용해 주문을 정의할 수 있게 만드는 방법
// 여러 빌더를 구현해야 한다.
// LambdaOrderBuilder 에서 빌더로 주문을 감쌀 Order 객체 order를 만든다.
// Consumer<람다빌더>를 받아 Order를 반환하는 메서드
// Consumer<람다빌더> 를 받는 buy, sell 메서드
// Consumer<람다빌더>와 Trade.type을 받는 trade 메서드를 가진다.

// TradeBuilder를 만든다.
// 양, 가격 setter를 만든다.
// stockBuilder를 저장하고 trade에 저장해줄 stock 메서드를 만든다.

// StockBuilder를 만든다.
// stock을 저장한다.
// symbol, market에 대한 setter를 만든다.
```
- 메서드 체인과 같이 플루언트 방식으로 주문을 정의할 수 있다.
- 중첩 함수 형식처럼 다양한 람다 표현식의 중첩수준과 비슷하게 도메인 객체의 계층구조를 유지한다.
- 잡음의 영향을 많이 받는다는 단점이 있다.

4. 조합하기
	- 배운 것들을 조합해서 아래와 같은 새로운 DSL을 개발할 수도 있다.
```java
Order order =  
forCustomer( "BigBank",
	buy( t -> t.quantity( 80 )
				.stock( "IBM" )
				.on( "NYSE" )
				.at( 125.00 )),
    sell( t -> t.quantity( 50 )
			    .stock( "GOOGLE" )
                .on( "NASDAQ" )
                .at( 125.00 )) );
```

 - 메서드 참조를 섞어 다른 간단한 기능을 추가할 수도 있다.
```java
public class TaxCalculator {
	public DoubleUnaryOperator taxFunction = d -> d;
	public TaxCalculator with(DoubleUnaryOperator f) {
		taxFunction = taxFunction.andThen(f);
    return this;
}
public double calculate(Order order) {  
	return taxFunction.applyAsDouble(order.getValue());
}

// main에서
double value = new TaxCalculator().with(Tax::regional)
						.with(Tax::surcharge)
						.calculate(order);
```

## 4. 실생활의 자바 8 DSL 
### 위의 내용 정리
- 메서드 체인
	- 장점
		- 메서드 이름이 키워드 인수 역할을 한다.
		- 선택형 파라미터와 잘 동작한다.
		- DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제할 수 있다.
		- 정적 메서드를 최소화하거나 없앨 수 있다.
		- 문법적 잡음을 최소화
	- 단점
		- 구현이 장황하다.
		- 빌드를 연결하는 접착 코드가 필요하다.
		- 들여쓰기 규칙으로만 도메인 객체 계층을 정의한다.
- 중첩 함수
	- 장점
		- 구현의 장황함을 줄일 수 있다.
		- 함수 중첩으로 도메인 객체 계층을 반영한다.
	- 단점
		- 정적 메서드의 사용이 빈번하다.
		- 이름이 아닌 위치로 인수를 정의한다.
		- 선택형 파라미터를 처리할 메서드 오버로딩이 필요하다.
- 람다를 이용한 함수 시퀀싱
	- 장점
		- 선택형 파라미터와 잘 동작한다.
		- 정적 메서드를 최소화하거나 없앨 수 잇다.
		- 람다 중첩으로 도메인 객체 계층을 반영한다.
		- 빌더의 접착 코드가 없다.
	- 단점
		- 구현이 장황하다.
		- 람다 표현식으로 인한 문법적 잡음이 존재한다.

### JOOQ
- SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어다.
```java
create.selectFrom(BOOK)
	.where(BOOK.PUBLISHED_IN.eq(2016))
	.orderBy(BOOK.TITLE)
```
- 스트림 API와 조합해 사용할 수 있다는 장점이 있다.

### 큐컴버
- 큐컴버 스크립팅 언어로 간단한 비즈니스 시나리오 정의
```java
   Feature: Buy stock
     Scenario: Buy 10 IBM stocks
       Given the price of a "IBM" stock is 125$
       When I buy 10 "IBM"
       Then the order value should be 1250$
```
- Given : 전제 조건 정의
- When : 시험하려는 도메인 객체의 실질 호출
- Then : 테스트 케이스의 결과를 확인하는 어설션(assertion)

### 스프링 통합
- 유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다.
	- 핵심 목표 : 복잡한 엔터프라이즈 통합솔루션 -> 단순한 모델, 비동기, 메시지 주도 아키텍처
 - 기반 애플리케이션 내의 경량의 원격, 메시징, 스케쥴링 지원
 - 채널, 엔드포인트, 폴러, 채널 인터셉터 등 메시지 기반의 애플리케이션에 필요한 가장 공통 패턴을 모두 구현한다.

# 스트림 소개
## 스트림이란 무엇인가?
- 자바 8 API에 새로 추가된 기능.
#### 특징
- 선언형 : 더 간결하고 가독성이 좋아진다.
- 조립할 수 있음 : 유연성이 좋아진다.
- 병렬화 : 성능이 좋아진다.

예시 코드
```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDishesName =
	menu.stream()
		.filter(d -> getCalories() < 400)
		.sorted(comparing(Dish::getCalories))
		.map(Dish::getName)
		.collect(toList());
		// stream() 대신 parallelStream()을 사용하여 병렬로 실행 가능
```

>스트림이란 `데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소`로 정의할 수 있다.

- 연속된 요소 : 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스 제공
- 소스 : 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
- 데이터 처리 연산 : 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다.

#### 스트림의 두 가지 중요 특징
- 파이프라이닝 :  스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
- 내부 반복 : 컬렉션과 같이 명시적으로 반복하지 않는다.

### 컬렉션과의 차이
- 데이터를 계산하는 시점
	- 컬렉션 : 요소들을 추가하기 전에
	- 스트림 : (이론적으로) 요청할 때만 요소를 계산
- 탐색할 수 있는 횟수
	- 컬렉션 : 원하는 만큼
	- 스트림 : 한 번
	- ```java
	List<String> title = Arrays.asList("Java8", "In", "Action");
	Stream<String> s = title.stream();
	s.forEach(System.out::println);
	s.forEach(System.out::println);
	// 두 번째 호출에서 IllegalStateException 발생 
    ```
- 데이터 반복 처리 방법
	- 컬렉션 : 외부 반복 - 사용자가 직접 요소를 반복
	- 스트림 : 내부 반복 - 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해준다.
		- 외부 반복에서는 병렬성을 스스로 관리해야 한다.

## 스트림 연산
```java
List<String> names = menu.stream()
						.filter(dish -> dish.getCalories() > 300)
						.map(Dish::getName)
						.limit(3)
						.collect(toList());
```
위 예제에서의 연산
- filter, map, limit : 서로 연결되어 파이프라인 형성. 비슷하게 sorted, distinct 등이 있다.
	- 중간 연산 : 연결할 수 있는 스트림 연산
- collect : 파이프라인을 실행한 다음에 닫음. 비슷하게 forEach, count 등이 있다.
	- 최종 연산 : 스트림을 닫는 연산

# 스트림 활용
## 1. 필터링
- 프레디케이트 필터링 : 프레디케이트를 인수로 받아서 필터링
- 고유 요소 필터링 : distint를 이용한 중복 필터링

## 2. 스트림 슬라이싱
- takeWhile : 조건에 만족할 때까지 while 처럼 반복하여 요소를 선택
- dropWhile : 조건에 만족할 때까지 while 처럼 반복하여 요소를 버린다.
	- 위 두가지 연산은 정반대의 작업을 수행한다.

**스트림 축소 : limit(n) **
**요소 건너뛰기 : skip(n)**
- filter와 같은 다른 연산과 함께 쓸 땐 순서에 주의해야 한다.
	- filter가 된 이후 skip을 하는 것과 이전에 skip을 하는 것은 많이 다르기 때문

## 3. 매핑
- map(함수나 메서드) : 요소들을 매개변수로 받은 함수를 거친 형태로 바꾸어 준다.
- flatMap(함수나 메서드) : 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑하여 평면화 해준다.

## 4. 검색과 매칭
### 매칭
1. anyMatch(프레디케이트) : 프레디케이트가 적어도 한 요소와 일치하는지 확인. 불리언을 반환하는 `최종 연산`
2. allMatch(프레디케이트) : 프레디케이트가 모든 요소와 일치하는지 검사
3. noneMatch(프레디케이트) : 모든요소가 프레디케이트와 불일치하는지 검사
- 위와 같은 연산들은 모든 검색이 끝나기 전에 조건에 따라 값을 반환할 수 있다.
	- 마치 && 나 || 와 같다.
### 검색
검색은 결과가 null 일 수 있으므로 `Optional<T>`를 반환한다.
Optional은 값이 없을 때 어떻게 처리할 지 강제하는 기능을 제공한다.
1. findFirst : 첫 번째 요소 찾기
2. findAny : 랜덤한 요소 찾기. 병렬화가 된 코드라면 첫 번째 요소를 찾는 것이 어려울 수 있는데, 순서가 상관없다면 제약이 적은 findAny를 사용하는 것이 좋다.

## 5. 리듀싱
- reduce(초기값, 함수)
- 모든 스트림 요소를 처리해서 값으로 도출한다.
- 함수형 프로그래밍 언어 용어로는 **폴드**라고 부른다.
### 요소의 합, 곱, 최대값, 최소값 구하기
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
int product = numbers.stream().reduce(1, (a, b) -> a * b);
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```
- 초기값을 받지 않도록 오버로드 된 reduce도 있지만 Optional 객체를 반환한다.

### stateful, stateless 스트림 연산
- stateful : reduce, sum, max. 결과를 누적할 내부 상태가 필요하다.
- stateless : map, filter. 입력 스트림에서 각 요소를 받아 

## 6. 실전 연습
## 7. 숫자형 스트림
1. 기본형 특화 스트림
- `IntStream`, `DoubleStream`, `LongStream`
```java
int calories = menu.stream()
					.mapToInt(Dish::getCalories) // IntStream 반환
					.sum();
```
- 기본형 특화 스트림이기 때문에 바로 sum()등의 특화된 함수를 사용 할 수 있다.
- IntStream 내부의 boxed 메서드를 통해 숫자 스트림을 (일반) 스트림으로 변환할 수 있다.

1-1. Optional 기본형 스트림
```java
OptionalInt maxCalories = menu.stream()
								.mapToInt(Dish::getCalories)
								.max();
int max = maxCalories.orElse(1); // 값이 없을 때 기본 값을 설정
```

2. 숫자 범위
```java
IntStream.rangeClosed(1, 100)
		.filter(n -> n % 2 == 0);
```

3. 숫자 스트림 활용 : 피타고라스 수
```java
Stream<int[]> pythagoreanTriples =
	IntStream.rangeClosed(1, 100).boxed()
			.flatMap(a -> // 아래 코드로 a 생성
					IntStream.rangeClosed(a, 100)
							.filter(b -> // 아래 코드 조건에 맞는 b 생성
								Math.sqrt(a*a + b*b) % 1 == 0)
								// 더한 값의 소수점 아래 수가 0이라면
							.mapToObj(b ->
								new int[]{a, b, (int)Math.sqrt(a*a + b*b)})));
// 스트림 완성
pythagoreanTriples.limit(5)
				.forEach(t ->
					System.out.println(t[0] + ", " + t[1] + ", " + t[2]));
// limit를 이용해 5개의 피타고라스 수 생성
```

## 8. 스트림 만들기
```java
// 값으로 스트림 만들기
Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
// 스트림 비우기
stream.empty();
//null이 될 수 있는 객체로 스트림 만들기
Stream<String> values = 
	Stream.of("config", "home", "user")
			.flatMap(key -> Stream.ofNullable(System.getProperty(key)));
// 배열로 스트림 만들기
int sum = Arrays.stream(new int[] {2, 3, 5, 7, 11, 13}).sum();
// 파일로 스트림 만들기
long uniqueWords = 0;
try(Stream<String> lines = 
		Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))) // 고유 단어 수 계산
						.distinct() // 중복 제거
						.count(); // 단어 스트림 생성
} catch(IOException e){
	// 파일 열다가 예외 발생 시 처리코드
}

// 함수로 무한 스트림 만들기 (언바운드 스트림)
// 1. Stream.iterate
Stream.iterate(0, n -> n + 1)
		.limit(10)
		.forEach(System.out::println); 
		// limit나 takeWhile등을 이용해 길이를 제한해 주어야 한다.
// 2. Stream.generate
Stream.generate(Math::random)
		.limit(5)
		.forEach(System.out::println);
		// generate는 값을 연속적으로 계산하지 않는다.
```

# 스트림으로 데이터 수집
## 1. 컬렉터란 무엇인가?
- Collectors 인터페이스 구현을 통해 스트림의 요소를 어떤 식으로 도출할 지 지정한다.
	- '각 요소를 리스트로 만들어라' 를 의미하는 toList를 구현하여 사용하는 등
```java
List<Transaction> transactions =
	transactionStream.collect(Collectors.toList());
	// 위의 Collectors.toList메서드를 어떻게 구현하느냐에 따라
	// 스트림에 어떤 리듀싱 연산을 수행할 지 결정된다!
```
- Collectors에서 제공하는 메서드의 기능은 크게
	1. 스트림 요소를 하나의 값으로 리듀스하고 요약
	2. 요소 그룹화
	3. 요소 분할
	로 구분할 수 있다.

## 2. 리듀싱과 요약
### 리듀싱
- 메뉴에서 칼로리가 가장 높은 요리를 찾는다고 가정
```java
Comparator<Dish> dishCaloriesComparator = 
	Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = 
	menu.stream()
		.collect(maxBy(dishCaloriesComparator));
```
### 요약 연산
```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
// summingInt는 객체를 int로 매핑한 컬렉터를 반환한다.

IntSummaryStatistics menuStatistics = 
	menu.stream().collect(summarizingInt(Dish::getCalories));
// IntSummaryStatistics{count=9, sum=4300, min=120,
//						average=477.7778, max= 800} 반환
```
### 문자열 연결
```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
// pork, beef, chicken, french fries, ... 하나의 스트링 값 반환
```
### 범용 리듀싱 요약 연산
```java
int totalCalories = menu.stream().collect(reducing(
						0, Dish::getCalories, (i, j) -> i+j));
```
- reducing(시작값, 변환함수, operator)를 받는다.
- reducing(operator)를 사용하는 때는 시작값 = 스트림 시작값, 변환함수 = 항등함수(자신을 그대로반환하는 함수) 이다
> reduce메서드를 collect 대신 사용하면 두 값을 하나로 도출하는 불변형 연산을 다르게 사용하는 것이므로 `의미론적인 문제`가 있다

## 3. 그룹화
```java
// groupingBy() 사용
Map<Dish.Type, List<Dish>> dishesByType =
		menu.stream().collect(groupingBy(Dish::getType));
// {FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]} 반환
// 다른 메서드가 연산 등을  통해 필요할 때 아래의 람다식을 groupingBy 파라미터로 사용해 줄 수 있다.
dish -> {
	if (dish.getCalories() <= 400) return CaloricLevel.DIET;
	else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
	else return CaloricLevel.FAT;
}
```

- 다만 위의 코드를 통해 grouping을 하면서 filter를 추가하고 싶을땐
```java
Map<Dish.type, List<Dish>> caloricDishesByType =
	menu.stream()
		.collect(groupingBy(Dish::getType,
				filtering(dish -> dish.getCalories() > 500,
				toList())));
```
- 위와 같이 필터를 따로 추가하는 것이 아니라 collect 내부에 넣어 오버로드된 groupingBy를 사용해야 한다.
- filter를 stream의 중간연산으로 사용할 때 **조건을 만족하는 요소가 없는 분류가 사라지는 문제**가 있다.
- Dish가 내부 요소로 List 등을 가질 땐, flatMapping 컬렉터를 이용하여 추출할 수 있다.

#### 다수준 그룹화
```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(

groupingBy(Dish::getType, //첫 번째 수준의 분류 함수
			   groupingBy(dish -> { // 두 번째 수준의 분류 함수
		if (dish.getCalories() <= 400) return CaloricLevel.DIET;
		else if (dish.getCalories() <= 700)  return CaloricLevel.NORMAL;
		else return CaloricLevel.FAT;
		})
	)
);
```
결과물은
`{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[prawns], NORMAL=[salmon]},  
OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}}` 와 같다.
- 외부 맵은 첫 번째 수준의 분류 함수에서 분류한 키 값 'fish, meat, other'를 갖는다.
- 외부 맵의 값은 두 번째 수준의 분류 함수의 기준 'normal, diet, fat'을 키값으로 갖는다.
<br>
- 아래코드는 collectingAndThen를 사용한다.
```java
Map<Dish.Type, Dish> mostCaloricByType = 
	menu.stream()
		.collect(groupingBy(Dish::getType,
				collectingAndThen(
					maxBy(comparingInt(Dish::getCalories)),
				Optional::get)));
```
- 위와 같이 collectingAndThen은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환한다.
- 리듀싱 컬렉터는 절대 Optional.empty()를 반환하지 않으므로 안전한 코드이다.

## 4. 분할
- 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능
- 그룹화 맵은 최대 두 개의 그룹으로 분류 된다. (true, false)
- partitioningBy 사용 예
```java
Map<Boolean, List<Dish>> partitionedMenu =
	menu.stream().collect(partitioningBy(Dish::isVegetarian));
// {false=[pork, beef, chicken, prawns, salmon],
// true=[french fries, rice, season fruit, pizza]} 반환
```
- 이제 `partitionedMenu.get(true)` 를 통해 모든 채식 메뉴를 얻을 수 있다.

#### 분할의 장점
- 분할함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 점
- 컬렉터를 두 가지 필터링 연산을 적용해서 조건에 맞는 리스트를 얻을 수도 있다.

#### 수를 소수와 비소수로 분할하기
```java
public boolean isPrime(int candidate) {
	int candidateRoot = (int) Math.sqrt((double)candidate);
	return IntStream.rangeClosed(2, candidateRoot)
	.noneMatch(i -> candidate % i == 0);
}
public Map<Boolean, List<Integer>> partitionPrimes(int n){
	return IntStream.rangeClosed(2, n).boxed()
										.collect(
			partitioningBy(candidate -> isPrime(candidate)));
}
```

## 5. Collector 인터페이스
```java
public interface Collector<T, A, R> {
	Supplier<A> supplier();  
	BiConsumer<A, T> accumulator(); Function<A, R> finisher();
	BinaryOperator<A> combiner();
	Set<Characteristics> characteristics();
}
// T는 수집될 스트림 항목의 제네릭 형식
// A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
// R은 수집 연산 결과 객체의 형식(항상 그런 것은 아니지만 대게 컬렉션 형식)이다.
```
#### supplier : 새로운 결과 컨테이너
```java
public Supplier<List<T>> supplier() {
       return () -> new ArrayList<T>();
} // 생성자 참조(ArrayList::new)를 전달하는 방법도 있다.
```
#### accumulator : 결과 컨테이너에 요소 추가
```java
public BiConsumer<List<T>, T> accumulator() {
       return (list, item) -> list.add(item);
}
```
#### finisher : 최종 변화값을 결과 컨테이너로 적용
```java
public Function<List<T>, List<T>> finisher() {
       return Function.identity();
} // 최종 결과로 반환하면서 누적 과정을 끝낼 때 호출할 함수를 반환
```
#### combiner : 두 결과 컨테이너 병합
```java
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```
#### Characteristics
- 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공한다.
- 다음 세 항목을 포함하는 열거형
	1. UNORDERED : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
	2. CONCURRENT : 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다.
	UNORDERED를 함께 설정하지 않았다면 데이터 소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 수행할 수 있다.
	3. IDENTITY_FINISH : 생략이 가능하다. 누적자 A를 결과 R로 안전하게 형변환 할 수 있다.
```java
// 위에서 살펴본 메서드들을 토대로 자신만의 ToListCollector를 구현할 수 있다.
import java.util.*;
import java.util.function.*;
import java.util.stream.Collector;
import static java.util.stream.Collector.Characteristics.*;

public class ToListCollector<T> implements Collector<T, List<T>, List<T>> { @Override

	public Supplier<List<T>> supplier() {  
		return ArrayList::new; // 수집연산의시발점
	}

	@Override  
	public BiConsumer<List<T>, T> accumulator() {
		return List::add; // 탐색한항목을누적하고바로누적자를고친다.
	}

	@Override
	public Function<List<T>, List<T>> finisher() {
		return Function.identity(); // 항등함수
	}
	@Override  
	public BinaryOperator<List<T>> combiner() {
	    return (list1, list2) -> {
	        list1.addAll(list2); 
	        return list1;
		};
	}
	@Override  
	public Set<Characteristics> characteristics() {
	// 컬렉터의플래그를IDENTITY_ FINISH,CONCURRENT로
	// 설정한다.
		return Collections.unmodifiableSet(EnumSet.of( IDENTITY_FINISH, CONCURRENT));
	}
}

// 메인 내부에서...
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
// 메뉴 스트림의 모든 요리를 수집하는 예제에 사용할 수 있다.
```

## 6. 커스텀 컬렉터를 구현해 성능 개선하기
- 예제는 소수점 만들기
```java
public class PrimeNumbersCollector implements Collector<Integer
			, Map<Boolean, List<Integer>>
			, Map<Boolean, List<Integer>>> {
	// 두 개의 빈 리스트를 포함하는 맵으로 수집 동작을 시작한다.
	@Override
	public Supplier<Map<Boolean, List<Integer>>> supplier() {
	    return () -> new HashMap<Boolean, List<Integer>>() {{
	        put(true, new ArrayList<Integer>());
	        put(false, new ArrayList<Integer>());
	    }};
	}
	
	@Override  
	public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
		acc.get( isPrime( acc.get(true), // 지금까지 발견한 소수 리스트를 isPrime메서드로 전달
		    candidate) )
		    .add(candidate); // 결과에 따라 맵에서 알맞은 리스트를 받아 현재 candidate를 추가
		};
	}
	
	@Override  
	public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
		return (Map<Boolean, List<Integer>> map1,
    Map<Boolean, List<Integer>> map2) -> {
    // 두 번째 맵을 첫 번째 맵에 병합한다.
            map1.get(true).addAll(map2.get(true));
            map1.get(false).addAll(map2.get(false));
        return map1;
		};
	}
	@Override  
	public Function<Map<Boolean, List<Integer>>,
	    Map<Boolean, List<Integer>>> finisher() {
			return Function.identity();
			// 최종 수집 과정에서 데이터 변환이 필요없다.
			// 항등 함수 반환
	}
	@Override
	public Set<Characteristics> characteristics() {
		return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));  
		// 순서에 의미가 있으므로 IDENTITY_FINISH만 해당
	}
}
```

# 병렬 데이터 처리와 성능
## 1. 병렬 스트림
```java
Stream.iterate(1L, i -> i + 1)
		.limit(n)
		.parallel() // 스트림을 병렬 스트림으로 변환할 수 있다.
		.reduce(0L, Long::sum);
		// .sequential()을 통해 병렬 스트림을 순차 스트림으로 변경할 수도 있다.
```
- 병렬 스트림은 내부적으로 ForkJoinPool을 사용한다.
- 기본적으로 ForkJoinPool은 프로세서 수에 상응하는 스레드를 갖는다.
- 기본형 스트림을 사용하면 성능이 더 좋아진다.

#### 병렬 스트림의 올바른 사용법
- **병렬 스트림이 올바로 동작하려면 공유된 가변 상태를 피해야 한다.**
	- sum과 같은 연산의 결과가 올바르게 나오지 않는다.

#### 병렬 스트림 효과적으로 사용하기
1. 확신이 서지 않으면 직접 측정하라. - 적절한 벤치마크를 통해
2. 박싱을 주의하라 - 자동 박싱 언박싱은 성능을 크게 저하시킬수 있는 요소이다.
3. 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다.
	- limit, findFirst처럼 요소의 순서에 의존하는 연산을 병렬로 수행 할 때
4.  스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라
	- 처리해야 할 요소 수가 N이고 하나의 요소를 처리하는 데 드는 비용을 Q라 하면 전체 스트림 파이프라인 처리 비용은 N * Q로 예상 할 수 있다.
5. 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.
	- 병렬화 과정에서 생기는 부가 비용을 상쇄할 수 있을 만큼의 이득을 얻지 못한다.
6. 스트림을 구성하는 자료구조가 적절한지 확인하라.
	- 분할을 하는 자료구조가 필요할 경우 ArrayList가 LinkedList보다 낫다.
7. 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
8. 최종 연산의 병합 과정 비용을 살펴보라

## 2. 포크 / 조인 프레임워크
- 포크 / 조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 서브태스크 각각의 결과를 합한다.
- 서브태스크를 스레드풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.
- `RecursiveTask<R>`의 서브클래스를 만들어 추상 메서드 `compute`를 구현하면 스레드 풀을 이용할 수 있다.
	- 해당 알고리즘은 분할 정복 알고리즘의 병렬화 버전이다.

## 3. Spliterator 인터페이스
- 분할할 수 있는 반복자 라는 의미
- Iterator와 비슷하지만 병렬 작업에 특화되어있다.
- 구조
```java
public interface Spliterator<T> {  
	boolean tryAdvance(Consumer<? super T> action);
	// 탐색할 요소가 남아있으면 true
	Spliterator<T> trySplit();
	// spliterator의 일부 요소를 분할하여 두번째 spliterator를 생성하는 메서드  
	long estimateSize();  
	// 탐색해야 될 요소 수 정보 제공
	int characteristics();
}
```
분할 과정에서 characteristics 메서드로 정의하는 Spliterator의 특성에 영향을 받는다.
- 특성
	- ORDERED : 순서에 유의
	- DISTINCT : x, y 두 요소를 방문했을 때 equals()는 항상 false
	- SORTED : 탐색된 요소는 미리 정의된 정렬 순서를 따름
	- SIZED : 크기가 알려진 소스로 Spliterator를 생성했다
	- NON-NULL : 모든 요소는 null이 아니다.
	- IMMUTABLE : 불변이다.
	- CONTURRENT : 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있다.
	- SUBSIZED : 해당 그리고 서브 Spliterator는 SIZED 특성을 갖는다.

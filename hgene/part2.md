# 2장 함수형 데이터 처리

---

### 1. 스트림 소개

- 컬렉션 : 데이터를 그룹화하고 처리.
- 컬렉션 연산 :
    - Java의 컬렉션 연산은 단점이 많이 존재 → 어떤 뜻인가?( Java vs SQL )
        - Java : 반복자, 누적자 등의 컨테이너 변수가 필요하고, 개발자가 직접 반복되는 연산을 처리해야함.
        - SQL : 질의를 어떻게 해야할 지 명시할 필요가 없음( 자동구현 ).
    - 그렇다면 많은 요소를 포함한 컬렉션 연산 성능을 높이기 위해서는? ***스트림*** 이 해결방안이 될 수 있음.
- 스트림 :
    - ***선언형*** 으로 컬렉션 연산 처리 ⇒ 가독성 확보.

      ** 선언형 : 데이터를 처리하는 임시구현코드 대신 질의로 표현.

    - 데이터를 투명하게 ***병렬*** 로 처리 ⇒ 성능 확보.
    - ***조립*** 할 수 있음( 빌딩 블록 ) ⇒ 유연성 확보.
- 컬렉션 vs 스트림


    |  | 컬렉션 | 스트림 |
    | --- | --- | --- |
    | 데이터 연산 시점 | 컬렉션에 추가되기 이전 | 사용자 요청 시 |
    | 탐색 횟수 | 1회 | 1회 |
    | 데이터 반복 방법 | 외부 반복 | 내부 반복 |
    | 병렬성 | 스스로 관리 | 선택시 자동 |

---

- 스트림 시작하기 :

  ** 스트림 : 데이터 연산을 지원하도록, 소스에서 추출된 연속적인 요소.

    - 연속적인 요소 :
        - 컬렉션 : 데이터에 초점( 시공간 복잡성과 관련된 데이터 저장 및 접근 ).
        - 스트림 : 계산에 초점( 표현 계산식 ).
    - 소스 : 정렬된 소스로 스트림 생성시, 정렬을 그대로 유지.
    - 데이터 연산 : 순차적 & 병렬적 연산 기능 제공.
    - ***파이프 라이닝*** : 자신을 반환하여 연산끼리 거대한 파이프라인 구성 ⇒ 최적화( 쇼트 서킷, 게으름 ) 제공.
    - ***내부 반복*** : 명시적인 반복문을 사용하지 않고, 라이브러리 내부에서 처리.
        - 외부 반복 : 소피아 장난감 있니? 있으면 담자! ( 없어질 때 까지 계속 반복 ).
        - 내부 반복 : 소피아 장난감 한번에 다 담자! ⇒ 작업을 병렬로 처리, 작업을 더 최적화된 순서로 처리.

---

- 스트림 연산
    - filter : 람다를 인수로 받아 특정 요소 제외.
    - map : 람다/메서드 참조를 인수로 받아 한 요소를 다른 요소로 변환 및 추출.
    - limit : 특정 개수 이상의 요소가 스트림에 저장되지 못하도록 개수 제한( 크기축소 ).
    - collect : 변환 방법을 인수로 받아 스트림에 누적된 데이터를 특정요소로 변환.

    ```java
    List<String> threeHighCaloricDishNames = menu.stream() // 메뉴에서 스트림 생성
    				.filter(dish -> dish.getCalories() > 300) // 고칼로리 요리 필터링
    				.map(Dish::getName) // 요리명 추출
    				.limit(3) // 선착순 3개만 선택
    				.collect(toList()); // 결과를 다른 리스트로 저장
    ```

- 스트림 연산 구분
    - 중간 연산 : 연결할 수 있는 스트림 연산.

    ```java
    Stream<T> filter(Predicate<T>)  : T -> boolean
    
    Stream<R> map(Function<T, R>)   : T -> R
    
    Stream<T> limit(int)
    
    Stream<T> sorted(Comparator<T>) : (T, T) -> int
    
    Stream<T> distinct() 
    ```

    - 최종 연산 : 스트림을 닫는 연산.

    ```java
    void forEach()        // 스트림의 각 요소를 소비하면서 람다를 적용
    
    long count()          // 스트림의 요소 개수를 반환
    
    Collections collect() // 스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션 생성
    ```

    - ***게으르다*** : 중간 연산을 모아뒀다가 최종 연산시 한꺼번에 처리.
    - ***최적화*** : 쇼트서킷, 루프 퓨전.

      ** 루프 퓨전 : 서로 다른 종류의 연산이 합쳐짐.


---

### 2. 스트림 활용

- ***필터링*** : 스트림 요소를 선택하는 방법.
    - Predicate 필터링 : Predicate을 인수로 받아서 일치하는 요소를 포함하는 스트림 반환.
    - 고유요소 필터링 : distinct 메서드를 스트림에서 지원.

    ```java
    /* Prediacte 필터링 */
    List<Dish> vegeterianMenu = menu.stream()
    			.filter(Dish::isVegeterian()) // 채식요리인지 여부 확인 메서드 참조
    			.collect(toList());
    
    /* 고유요소 필터링 */
    List<Integer> evenNumbers = numbers.stream()
    			.filter(i -> i % 2 == 0)
    			.distinct()
    			.collect(toList());
    ```


---

- ***슬라이싱*** : 스트림의 요소를 선택하거나, 스킵하는 방법.
    - Predicate 슬라이싱 : 스트림이 제공하는 takeWhile, dropWhile 메서드 활용.
        - 필터링 vs 슬라이싱
            - 필터링 : 스트림 요소를 반복하며 각 요소에 기준 적용.
            - 슬라이싱 : 이미 정렬되어 있는 경우, 조건에 부합하지 않으면 반복 중단.

    ```java
    /* 아래의 두 결과는 서로 반대 */
    List<Dish> sliceMenu = specialMenu.stream()
    			.takeWhile(dish -> dish.getCalories() < 320)
    			.collect(toList());
    
    List<Dish> sliceMenu = specialMenu.stream()
    			.dropWhile(dish -> dish.getCalories() < 320)
    			.collect(toList());
    ```


---

- ***스트림 축소*** : limit 메서드를 통해 주어진 값 이하의 크기를 갖는 스트림 반환.

    ```java
    List<Dish> sliceMenu = specialMenu.stream()
    			.takeWhile(dish -> dish.getCalories() < 320)
    			.limit(3) // 조건에 부합하는 요소 3개가 모이면 중단
    			.collect(toList());
    ```


---

- ***요소 건너뛰기*** : skip 메서드를 통해 처음 N개 요소를 제외한 스트림 반환.
    - limit & skip 은 서로 상호보완적인 역할 수행.

    ```java
    List<Dish> sliceMenu = specialMenu.stream()
    			.takeWhile(dish -> dish.getCalories() < 320)
    			.skip(2) // 조건에 부합하는 2개의 요소를 처음에 건너뜀
    			.collect(toList());
    ```


---

- ***매핑*** : map, flatMap 메서드를 통해 특정 데이터를 선택하는 기능( 변환 ).
    - map : 인수로 제공된 함수를 적용한 결과를 스트림으로 반환.
    - flatMap : 스트림의 각 값을 스트림으로 만든 다음, 모든 스트림을 하나의 스트림으로 연결( 평면화 ).

    ```java
    /* 아래의 words 배열에 포함된 고유문자( 중복X )를 포함하는 리스트 만들기 */
    List<String> words = Arrays.asList("Hello", "World");
    
    /* map :
     * Stream<String[]> { { "H","e","l","l","o" },{ "W","o","r","l","d" } }
     */
    List<String[]> uniqueCharacters = words.stream()
    				.map(word -> word.split("")) // 각 단어를 개별 문자를 포함하는 배열로 변환
    				.distinct() // 두 원소를 가진 배열 중 고유한 값 걸러내기
    				.collect(toList());
    
    /*
     * flatMap :
     * List<String> { "H","e","l","o","W","r","d" }
     */
    List<String> uniqueCharacters = words.stream()
    				.map(word -> word.split("") // 각 단어를 개별 문자를 포함하는 배열로 변환
    				.flatMap(Arrays::stream) // 생성된 스트림을 하나의 스트림으로 평면화
    				.distinct() // 전체 알파벳을 가진 요소들 중 고유한 값 걸러내기
    				.collect(toList());
    ```


---

- ***검색과 매칭*** : 특정 속성이 데이터 집합에 있는지 여부 검색하는 최종연산 ⇒ 쇼트서킷으로 최적화.
    - anyMatch() : Predicate가 적어도 한 요소와 일치하는지 확인.
    - allMatch() : Predicate가 모든 요소와 일치하는지 확인.
    - noneMatch() : Predicate가 모든 요소와 일치하지 않는지 확인.

    ```java
    if(menu.stream().anyMatch(Dish::isVegeterian)) System.out.println("vege");
    
    boolean isHealthy = menu.stream()
    					.allMatch(dish -> dish.getCalories() < 1000);
    
    boolean isHealthy = menu.stream()
    					.noneMatch(d -> d.getCalories() >= 1000);
    ```


---

- ***Optional*** : null 예외 회피 방법, 아무런 값도 반환하지 않을 가능성이 있는 스트림 메서드와 함께 사용가능.
    - isPresent() : 현재 값의 존재여부 반환.
    - ifPresent() : 값 존재시 동작 수행.
    - get() : 값이 존재하면 반환, 없으면 예외 발생.
    - orElse() : 값이 존재하면 반환, 없으면 지정한 값 반환.

---

- ***요소 검색*** : 특정 요소 반환하는 최종 연산, Optional을 통해 null 처리.
    - findAny() : 임의의 요소 반환, 논리적 순서 상관 X ⇒ 병렬스트림 제약 감소.
    - findFirst() : 첫번째 요소 반환, 논리적 순서 상관 O ⇒ 병렬스트림 제약 증대.

    ```java
    menu.stream()
    	.filter(Dish::isVegeterian)
    	.findAny() // 요소 검색
    	.ifPresent(dish -> System.out.println(dish.getName()); // Optional
    ```


---

- ***리듀싱( 폴드 )*** : 모든 스트림 요소를 처리해서 값으로 도출.
    - 반복된 패턴 추상화 ⇒ 병렬화 용이( 반복적인 누적자 패턴에서는 가변 변수 공유로 인해 병렬이 어려움 ).
    - reduce(초깃값, BinaryOperator<T>)
    - reduce(BinaryOperator<T>) : 초깃값이 없을 에 따라 null 가능하기 때문에 Optional 로 처리.

    ```java
    int sum = numbers.stream().reduce(0, Integer::sum);
    
    Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
    
    Optional<Integer> max = numbers.stream().reduce(Integer::max);
    
    Optional<Integer> min = numbers.stream().reduce(Integer::min);
    ```


---

- 스트림 연산
    - 내부 상태를 갖는 연산 : 다음 요소에 대한 연산을 위해 이전 연산의 결과가 필요함( 내부 상태의 크기는 한정되어 있음).
    - 내부 상태를 갖지 않는 연산.

---

- 중간 연산과 최종 연산 정리

    ```java
    /* 중간 연산 */
    Stream<T> filter(Predicate<T>) : T -> boolean
    
    Stream<T> distinct(); // 상태가 있는 언바운드
    
    Stream<T> takeWhile(Predicate<T>) : T -> boolean
    
    Stream<T> dropWhile(Predicate<T>) : T -> boolean
    
    Stream<T> skip(long) // 상태가 있는 바운드
    
    Stream<T> limit(long) // 상태가 있는 바운드
    
    Stream<R> map(Function<T, R>) : T -> R
    
    Stream<R> flatMap(Function<T, Stream<R>>) : T -> Stream<R>
    
    Stream<T> sorted(Comparator<T>) : (T, T) -> int // 상태가 있는 언바운드
    
    /* 최종 연산 */
    boolean anyMatch(Predicate<T>) : T -> boolean
    
    boolean noneMatch(Predicate<T>) : T -> boolean
    
    boolean allMatch(Predicate<T>) : T -> boolean
    
    Optional<T> findAny()
    
    Optional<T> findFirst()
    
    void forEach(Consumer<T>) : T -> void
    
    R Collect(Collector<T, A, R>
    
    Optional<T> reduce(BinaryOperator<T>) : (T, T) -> T // 상태가 있는 바운드
    
    long count()
    ```


---

- ***기본형 특화 스트림***
    - 기본형 특화 스트림이 아닌 스트림에는 박싱 비용 존재.
    - 숫자 스트림을 효율적으로 연산할 수 있도록 함( 박싱 과정의 효율성 증대 ).
    - IntStream, DoubleStream, LongStream 3가지 특화 스트림 존재.
    - mapToInt, mapToDouble, mapToLong : 스트림 → 특화 스트림.
    - sum, max, min, man, average 등 다양한 유틸리티 메서드 존재.
    - boxed : 특화 스트림 → 스트림.
    - OptionalInt를 통해 값이 없는 상황에서 반환할 기본형 값 지정 가능.

    ```java
    int calories = menu.stream()
    			.mapToInt(Dish::getCalories) // IntStream 반환
    			.sum(); // IntStream이 제공하는 유틸리티 메서드
    
    Stream<Integer> calories = menu.stream()
    			.mapToInt(Dish::getCalories) // IntStream 반환
    			.boxed(); // 다시 Stream 반환
    
    // 스트림에 요소가 없는 상황( IntStream의 기본값이 0 ), 실제 최댓값이 0인 상황을 구분
    OptionalInt maxCalories = menu.stream()
    			.mapToInt(Dish::getCalories)
    			.max();
    ```


---

- ***숫자 범위*** : IntStream, LongStream에서 제공하는 range, rangeClose 메서드를 통해 시작값( 1st 인수 ) ~ 종료값( 2nd 인수 ) 범위의 요소 반환.
    - LongStream : 종료값 포함 X.

    ```java
    // [1, 100] 의 범위 지정
    IntStream evenNumbers = IntStream.rangeClosed(1, 100);
    ```


---

- 값으로 스트림 만들기
    - Stream.of() : 인수로 받은 값들을 요소로 한 스트림 생성.
    - Stream.empty() : 빈 스트림 생성.
    - Stream.ofNullable() : null인 값이 인수라면, empty(). null이 아닌 값이 인수라면, of().
    - Arrays.stream() : 배열을 인수로 받아 스트림 생성.
    - Files.line() : 파일의 각 행을 문자열로 반환받아 스트림으로 생성 가능.
    - Stream.iterate() : 함수에서 스트림을 만들어 무한 스트림 생성.
    - Stream.generate() : Supplier<T>를 인수로 받아 나중에 연산에 사용할 값 저장 X → 연속적연산 X.

    ```java
    Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
    
    Stream<String> emptyStream = Stream.empty();
    
    Stream<String> nullableStream = Stream.ofNullable(System.getProperty("A");
    
    int[] numbers = { 2,3,5,7,11,13 };
    int sum = Arrays.stream(numbers).sum();
    
    long uniqueWords = 0;
    try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset()) {
    	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
    		.distinct()
    		.count();
    } catch (IOException e) {
    	e.printStackTrace();
    }
    
    Stream.iterate(0, n -> n+2)
    			.limit(10) // limit, takewhile 과 함께 사용되어 범위 제한
    			.forEach(System.out::println);
    
    Stream.generate(Math::random)
    			.limit(5) // limit, takewhile 과 함께 사용되어 범위 제한
    			.forEach(System.out::println);
    ```


---

### 3. 스트림으로 데이터 수집

- ***collect()*** : 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산 수행 가능.
    - collect(Collectors.method()) vs Stream().method()
        - 가변 컨테이너 작업이면서 병렬성을 확보하려면 collect 메서드로 구현하는 것이 바람직.
        - 높은 수준의 추상화, 커스터마이징 용이.
        - but 코드가 복잡함.
- ***Collectors*** : 다양한 요소 누적 방식이 저장된 인터페이스 ⇒ 최종 연산시 한꺼번에 처리.

---

- ***리듀싱 & 요약*** : 스트림 요소를 하나의 값으로 리듀스 연산 및 요약.
    - counting() : 요소 개수.
    - maxBy(), minBy() : Comparator를 인수로 받아 최댓값, 최솟값 반환.
    - summingInt(), summingDouble(), summingLong() : 합계.
    - averagingInt(), averagingDouble(), averagingLong() : 평균.
    - joining() : 각 객체에 toString()을 호출하여 모든 문자열을 하나의 문자열로 연결하여 반환.
    - summarizingXX() : XX형SummaryStatics에 모든 정보( count,sum,max,min,average ) 수집.
    - reducing() : 범용 리듀싱 요약 연산( 가독성은 좋지 않음 ).

    ```java
    long cntDishes = menu.stream().collect(Collectors.counting());
    
    Comparator<Dish> dishComparator = Comparator.comparingInt(Dish::getCalories);
    Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishComparator));
    
    double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
    
    IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
    
    String shortMenu = menu.stream().map(Dish::getName).collect(joining());
    
    int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i,j) -> i+j));
    ```


---

- ***요소 그룹화*** : 데이터 집합을 하나 이상의 특성으로 분류해서 그룹화.
    - groupingBy() : 인수를 분류 함수로 받음.

      ** 분류 함수 : 단순하면 메서드 참조로, 복잡하면 람다로.

    - 그룹화 결과에 대한 조작
        - Collector 내부에 filtering 함수를 인수로 넣음.
        - flatMapping, mapping 함수를 인수로 넣음.
    - 다수준의 그룹화
        - 바깥쪽 grouping 메서드 안에 항목을 분류할 두번째 grouping 메서드를 전달.
        - 이때, 그룹화에 대한 연산 수행시, 어차피 존재하지 않는 키는 애초에 생성되지 않으므로, Optional 사용할 필요 X.
            - collectingAndThen() 메서드를 통해 연산결과를 다른 방식으로 활용 가능.

    ```java
    Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType);
    
    Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
    			.collect(groupingBy(dish -> 
    				if (dish.getCalories() <= 400) return CaloricLevel.DIET;
    				else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
    				else return CaloricLevel.FAT;
    			));
    
    Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream()
    			.collect(groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList())));
    
    Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByCaloricLevel = menu.stream().collect(
    	groupingBy(Dish::getType, groupingBy(dish -> {
    			if (dish.getCalories() <= 400) return CaloricLevel.DIET;
    			else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
    			else return CaloricLevel.FAT;
    	}))
    );
    
    Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
    			.collect(groupingBy(Dish::getType, collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
    ```


---

- ***요소 분할*** : 분할 함수를 분류함수로 사용하는 특수한 그룹화 기능.

  ** 분할 함수 : Boolean 값을 반환하는 함수.

    - 결과로 최대 참 or 거짓 2개의 그룹을 분류.
    - partitioningBy() : 컬렉터를 두번째 인수로 전달해서 해당 기준으로 요소 분할.

    ```java
    Map<Boolean, List<Dish>> partitionedMenu = menu.stream()
    			.collect(partitioningBy(Dish::isVegeterian));
    
    Map<Boolean, Map<Dish.Type, List<Dish>>> vegeterianDishesByType = menu.stream().collect(
    					partitioningBy(Dish::isVegeterian, groupingBy(Dish::getType))
    );
    ```


---

- Collectors 클래스의 정적 팩토리 메서드

    ```java
    List<T> toList() // 스트림의 모든 항목을 리스트로 수집
    
    Set<T> toSet() // 스트림의 모든 항목을 중복이 없는 집합으로 수집
    
    Collection<T> toCollection() // 스트림의 모든 항목을 발행자가 제공하는 컬렉션으로 수집
    Collection<Dish> dishes = menuStream.collect(toCollection(), ArrayList::new);
    
    Long counting()
    
    Integer summingInt()
    
    Double averagingInt()
    
    IntSummaryStatistics summarizingInt()
    
    String joining()
    
    Optional<T> maxBy()
    
    Optional<T> minBy()
    
    T reducing()
    
    T collectingAndThen()
    
    Map<K, List<T>> groupingBy()
    
    Map<Boolean, List<T>> partitioningBy()
    ```


---

- Collectors 구성
    - Collector<T, A, R>

        ```java
        public interface Collector<T, A, R> {
        	Supplier<A> supplier();
        	BiConsumer<A, T> accumulator();
        	Function<A, R> finisher();
        	BinaryOperator<A> combiner();
        	Set<Characteristics> characteristics();
        }
        ```

        - T : 수집될 스트림 항목.
        - A : 중간 연산 결과 저장.
        - R : 결과.
    - supplier() : 수집 과정에서 빈 누적자 인스턴스 생성 및 반환.
    - accumulator() : 리듀싱 연산을 수행하는 함수 반환.
    - finisher() : 스트림 탐색을 끝내고, 누적자 객체를 최종결과로 변환하면서 누적과정을 끝낼때 호출할 함수 반환.
    - combiner() : 스트림의 서로 다른 병렬 파트를 처리할 때, 누적자가 이 결과를 어떻게 처리할 지 정의.
    - characteristics() : 병렬로 처리할 것인지, 그렇다면 어떤 최적화를 할 것인지에 대한 힌트 제공.
        - UNORDERED : 순서 영향 X.
        - CONCURRENT : 다중 스레드 환경에서 accumulator 동시 호출 가능( 순서가 무의미한 상황에만 사용가능).
        - IDENTITY_FINIFH : 리듀싱 과정의 최종 결과로 누적자 객체 바로 사용 가능.

---

- ***커스텀 Collector 생성*** : 위의 Collector 구성요소를 정의 및 구현해야 함.
    - 성능 향상은 가능하지만, 가동성 및 재사용성이 떨어질 가능성이 존재.

    ```java
    public class ToListCollector<T> implements collector(T, List<T>, List<T>) {
    	@Override
    	public Supplier<List<T>> supplier() {
    		return ArrayList::new; // 수집 연산의 시작점
    	}
    
    	@Override
    	public BiConsumer<List<T>, T> accumulator() {
    		return List::add; // 탐색한 항목에 대한 리듀싱 연산 & 결과 누적
    	}
    
    	@Override
    	public Function<List<T>, List<T>> finisher() {
    		return Function.identity(); // 항등 함수
    	}
    
    	@Override
    	public BinaryOperator<List<T>> combiner() {
    		return (list1, list2) -> {
    			list1.addAll(list2);
    			return list1;
    		}
    	}
    
    	@Override
    	public Set<Characteristics> characteristice) {
    		return Collections.unmodifiableSet(
    			EnumSet.of(IDENTITY_FINISH, CONCURRENT)
    		);
    	}
    }
    ```


---

### 4. 병렬 데이터 처리와 성능

- ***병렬 스트림***
    - parallelStream() : stream() 대신 사용하여 병렬 스트림 생성.
    - 모든 멀티코어 프로세서가 각각의 청크를 처리할 수 있도록 할당.
    - pararell() : 이후 연산을 병렬로 처리.
    - sequential() : 이후 연산을 순차적으로 처리.
        - 두 메서드 중 최종적으로 호출된 메서드가 파이프라인에 영향을 미침.
- 성능 비교
    - 반복
    - 스트림 순차 : 반복에 비해 오히여 박싱 비용 증가할 수 있음 → 숫자형 특화 스트림 사용으로 해소.
    - 스트림 병렬 : 순차에 비해 오히려 스레드 생성 및 할당 비용 증가 가능 → 범위지정을 통해 해소.
- 병렬 스트림을 사용하려면?
    - 멀티코어 간 데이터 이동 비용보다 훨씬 큰 연산만 병렬로 처리할 것.
    - 공유된 가변 상태가 없을 때만 병렬로 처리할 것.
    - 순서에 의존하지 않는 경우에만 병렬로 처리할 것.

---

- ***fork / join 프레임워크***
    - 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음, 서브태스크 결과를 합쳐서 전체 결과 도출.
    - ExcutorService 구현 : 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당.
    - RecursiveTask<R>의 compute() 구현 :
        - 태스크를 서브태스크로 분할.
        - 더이상 분할할 수 없을 때, 서브태스크의 결과를 생산할 알고리즘 정의( 분할 정복 ).

---

- ***작업 훔치기*** : 작업이 먼저 끝난 스레드가 다른 스레드의 작업을 가져와서 처리 ⇒ 모든 스레드의 부하를 비슷하게 유지 가능.

---

- ***Spliterator 인터페이스*** : 스트림을 자동으로 분할 및 탐색해야할 데이터를 어떻게 분할할 것인지 정의.

```java
public interface Spliterator<T> {
	boolean tryAdvance(Consumer<> super T> action);
	Spliterator<T> trySplit();
	long estimateSize();
	int characteristics();
}
```

- tryAdvance(Consumer<> super T> action) : 탐색할 요소가 남아있는지 여부 반환.
- trySplit() : 자신이 반환한 요소를 분할해서 다음 탐색할 요소 생성.
- estimateSize() : 탐색할 요소 수.
- characteristics() : Spliterator를 더 잘 제어하고, 최적화할 수 있도록 도움.
    - ORDERED : 리스트처럼 요소에 정해진 순서가 있으므로, Spliterator는 요소를 탐색하고 분할할 때 이 순서에 유의해야 함.
    - DISTINCT : x, y 두 요소를 방문했을 때, x.equals(y)는 항상 false를 반환.
    - SORTED : 탐색된 요소는 미리 정의된 순서를 따름.
    - SIZED : 크기가 알려진 소스로 Spliterator를 생성했으므로 estimatedSize()는 정확한 값을 반환.
    - NON-NULL : 탐색하는 모든 요소는 null이 아님.
    - IMMUTABLE : 이 Spliterator의 소스는 불변( 탐색하는 동안, 요소 추가, 수정, 삭제 불가 ).
    - CONCURRENT : 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있음.
    - SUBSUZED : 이 Spliterator 그리고, 분할되는 모든 Spliterator는 SIZED 특성을 가짐.

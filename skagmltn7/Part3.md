<h1>Part3. 스트림과 람다를 이용한 효과적 프로그래밍</h1>
<br>

<h2>1. 컬렉션 API 개선</h2>

<h3>컬렉션 팩토리</h3>

- 적은 요소를 원하는 컬렉션 객체로 쉽게 만들어 주는 방법
- 크기는 고정되어 있기 때문에 갱신O, 추가, 삭제X
- 컬렉션에 요소를 변경(수정, 추가, 삭제)할 경우 `UnsupportedOperationException` 예외가 발생 -> **컬렉션이 의도치 않게 변하는 것을 방지**
- 데이터 처리 형식을 성정하거나 데이터 변환할 필요가 있을 경우, Collectors.method()권장
    - 오버로드<br>
       10개 미만의 요소의 경우 오버로딩된 메소드가 사용
        
        ```java
        static <E> List<E> of(E e1, E e2, E e3, E e4)
        static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)
        ```
        
    - 가변 인수<br>

        ```java
        static <E> List<E> of(E... elements)
        ```
        `추가 배열 할당` -> `리스트로 감쌈` -> 추가 배열를 없애기 위해 `가비지 컬렉션 사용` 비용 지불
<br><br>

<h4>배열 팩토리</h4>

- `Arrays.asList()` 팩토리 메소드를 사용하여 고정된 크기의 리스트를 만들 수 있음.
- 요소 변경O, 추가, 삭제X

```java
List<String> AlphabetList = List.of("a","b","c","d");// {"a","b","c","d"}
Alphabet.set(0,"A"); // {"A","b","c","d"}
Alphabet.add("e"); // UnsupportedOperationException 발생
Alphabet.remove(0); // UnsupportedOperationException
```

<h4>리스트 팩토리</h4>

- `List.of()` 팩토리 메소드를 사용하여 리스트를 만들 수 있음.
- 요소 변경, 추가, 삭제 X

```java
List<String> AlphabetList = List.of("a","b","c","d"); // {"a","b","c","d"}
Alphabet.set(0,"A"); // UnsupportedOperationException 발생
Alphabet.add("e"); // UnsupportedOperationException 발생
Alphabet.remove(0); // UnsupportedOperationException
```

<h4>집합 팩토리</h4>

- `Set.of()` 팩토리 메소드를 사용하여 집합을 만들 수 있음.
- 매개 변수로 중복된 값을 넣을 수 없음
- 추가, 삭제X

```java
Set<String> AlphabetSet1 = Set.of("a","b","c","d"); // {"a","b","c","d"}
Set<String> AlphabetSet2 = Set.of("a","b","a"); // IllegalArgumentException 발생
```

<h4>맵 팩토리</h4>

- `Map.of()`, `Map.ofEntries()` 팩토리 메소드를 사용하여 맵을 만들 수 있음.
- 변경, 추가, 삭제X

```java
Map<String,Integer> AlphabetMap1 = Map.of("a",0,"b",1,"c",2,"d",3);
Map<String,Integer> AlphabetMap2 = Map.ofEntries( // entry 추가 객체 할당
												Map.entry("a",0),
												Map.entry("b",1),
												Map.entry("c",2),
												Map.entry("d",3)
												);
AlphabetMap2.put("a", 9); // UnsupportedOperationException 발생
AlphabetMap2.remove("a"); // UnsupportedOperationException 발생
AlphabetMap2.replace("a", 90); // UnsupportedOperationException 발생
```

<h3> 리스트와 집합 처리 </h3>

- List, Set 인터페이스에 추가된 메소드
- 스트림은 새로운 결과를 바내환하지만 다음 메소드들은 `기존 컬렉션 자체를 바꿈`
- 컬렉션를 바꾸는 데에 있어 에러와 복잡함을 줄이기 위함

<h4> removeIf() </h4>

- List나 Set를 구현하거나 상속한 경우 사용 가능
- Predicate를 만족할 경우 제거

```java
numbers.removeIf(num -> num < 3) // 3보다 작을 경우 리스트에서 숫자 삭제

```
<h4> replaceAll </h4>

- List에서 제공하는 기능
- UnaryOperator를 사용하여 요소 변경

```java
numbers.replaceAll(num -> num+10) // 리스트에 있는 모든 수에 10 더하기

```
<h4> sort </h4>

- List에서 제공하는 기능

<h3> 맵 처리 </h3>

- 자바8 이후 추가된 디폴트 메소드

<h4>forEach</h4>

- Map.Entry<K,V>에 접근하여 항목 집합 반복

```java
AlphabetMap2.forEach((key,value) -> System.out.println(key + "is" + value + "th number"));
```

<h4>정렬메서드</h4>

- `Entry.comparingByValue`, `Entry.comparingByKey`를 사용하여 값, 키를 기준으로 정렬할 수 있음

```java
AlphabetMap2.entrySet()
                .stream()
                .sorted(Entry.comparingByKey()) // 알파벳순으로 정렬
                .forEachOrdered(System.out::println); 

```

<h4>getOrDefault</h4>

- 키가 맵에 존재하지 않을때 처리하는 방법
- 기존의 get()은 키가 맵에 존재 하지 않을 경우 NullPointerException 예외 발생, 이를 방지하기 위한 메소드 
- `getOrDefault(찾고자하는 키, 존재하지 않을 경우 반환할 값)`
- 해당 키의 value가 null일 경우엔 null반환

```java
AlphabetMap2.getOrDefault("a",-1) // 0
AlphabetMap2.getOrDefault("e",-1) // -1
```

<h4>계산 패턴</h4>

- 키가 존재 여부에 따라 실행할 동작을 결정

- `computeIfAbsent`
    - 키가 없거나 null일 경우, 키를 사용하여 새 값 계산 후 맵 추가 
    - computeIfAbsent(찾고자 하는 키, 없을 경우 실행할 동작)
<br>

- `computeIfPresnet`
    - 키가 있을 경우 새 값 계산후 맵 추가
    - null반환시 맵에서 제거
    - computeIfPresnet(찾고자 하는 키, 있을 경우 실행할 동작)
<br>

- `compute`
    - 제공된 키로 새 값 계산 후 맵 추가
<br>

<h4>삭제 패턴</h4>

- 오버로드된 `remove(key,value)`로 특정 원하는 값을 삭제해 주는 메소드
<br>

<h4>교체 패턴</h4>

- `replaceAll`
    - Bifunction로 요소 변경

    ```java
    AlphabetMap2.replaceAll((key,value) -> key.toUpperCase()); // 키값으로 소문자가 대문자로 변경
    ```
<br>

- `Replace`
    - 키가 존재할 경우 값 변경
    - key, value가 같아야지만 변경하는 버전도 있음
<br>

<h4>합침</h4>

- `putAll`
    - 중복된 키가 없을 경우 두 개의 Map을 합침
<br>

- `merge`
    - 중복된 키가 있을 경우 BiFunction으로 어떻게 처리할 것인지 결정함

    ```java
    AlphabetMap1.forEach((k,v)-> AlphabetMap2.merge(k, v, (v1,v2)-> v1)); // Map1과 Map2의 키 값이 같으면 Map1의 값으로 맵 합침
    ```

    - 지정된 키의 값이 없거나 null이면 값을 제거하거나 새로운 값으로 매핑

    ```java
    AlphabetMap1.merge("e",0L,(key,value)->key.charAt(0)-'a'); // 해당 키값이 없을 경우 알파벳 순서를 넣어줌
    ```
<br>

<h3>개선된 ConcurrentHashMap</h3>

- 상태를 잠그지 않고 연산 수행
    - 바뀔 수 있는 객체, 값, 순서에 의존하지 않는 연상 함수를 사용해야함
- 병렬성 기준값 지정
    - 맵의 크기 < 기준값 : 순차적 연산 실행
    - 기준값==1 : 병렬성 극대화
    - 기준값==Long.MAX_Value : 한 개의 스레드 사용
- `forEach`<br>
: 각 (key, value) 쌍에 액션 실행
- `reduce`<br>
: 모든 (key, value) 쌍에 리듀스 함수 실행후 값 함침
- `search`<br>
: 널이 나올 경우 각 (key, value)함수 적용 중단

키, 값으로 다음 형태를 연산을 지원함

||forEach|reduce|search|
|:----:|:----:|:----:|:----:|
|(key,value)|forEach|reduce|search|
|key|forEachKey|reduceKeys|searchKeys|
|value|forEachValue|reduceValues|searchValues|
|Entry객체|forEachEntry|reduceEntries|searchEntries|

<br>

- 위의 메소드를 사용하는 경우 size보다 `mappingCount()`권장
    - 매핑의 개수가 int범위를 넘어서는 경우 대처 가능
- ConcurrentHashMap을 집합 뷰로 반환하는 `keySet` 메서드를 제공
    - 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받음
    - newKeySet이라는 메서드를 통해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있음
<br>

<h2>2. 리팩터링, 테스팅, 디버깅</h2>

<h3>가독성과 유연성을 개선하는 리팩터링</h3>

<h4>코드 가독성 개선</h4>

- `익명 클래스 -> 람다표현식`
    ||this|shdow|인스턴스화|
    |:----:|:----:|:----:|:----:|
    |익명 클래스|익명 클래스 자신|O|명시적으로<br> 형식정해짐|
    |람다 표현식|람다를 감싸는 클래스|X|콘텍스트에 의존<br>-><br>명시적 형변환으로 해결가능|

***shdow : 자신 코드 블럭안에 있으면 밖에 있는 같은 이름의 변수는 무시 <br>
e.g. 같은 이름의 지역변수와 인스턴스 변수가 있는경우 가까운 지역변수를 사용하는 방식
<br>
***인스턴스화 : 같은 시그니처를 갖은 함수형 인터페이스인 경우 어떤 인터페이스를 사용할지 람다의 경우 고르기 어려움<br>
e.g. 다중상속을 허용할 경우 조상클래스에 같은 이름의 메소드가 있을 경우 어느 메소드를 써야할지 모호함

- `람다 표현식 -> 메서드 참조`
- `명령형 데이터 처리 -> 스트림`

<h4>코드 유연성 개선</h4>

- 람다 표현식을 이용할 경우 동작 파라미터화를 쉽게 구현 가능, 이를 위해 `함수형 인터페이스 적용`

    - `조건부 연기 실행`<br>
    불필요한 if문 제거, 코드 가독성↑, 캡슐화

    - `실행 어라운드`<br>
    반복적인 준비, 종료과정을 람다로 변환

<h3>람다로 객체지향 디자인 패턴 리팩터링하기</h3>

- **전략**<br>
    - 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 전략(알고리즘)을 선택
- **템플릿 메서드**<br>
    - 이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는 상황에 적합
- **옵저버**<br>
    - 어떤 이벤트가 발생했을 때 한 객체(subject)가 다른 객체들(observer)에 자동으로 알려줘야하는 경우 적합
    ```java
    interface Observer {
        void notify(String tweet);
    }
    ```

- **의무 체인**<br>
    - 작업 처리 객체의 체인(동작 체인 등)을 만드는 경우
    - 한 객체가 어떤 작업을 처리 -> 다른 객체로 결과를 전달 -> 다른 객체도 해야 할 작업을 처리 -> 또 다른 객체로 전달

- **팩토리**<br>
    - 인스턴스화 로직을 클라이언트에게 노출하지 않고 객체 생성하는 경우

<h3>람다 테스팅</h3>

- 람다를 직접적으로 테스팅하기는 어려움
- 람다는 익명함수이기 때문에 람다 자체를 직접 호출하여 테스트X
- 람다 테스팅 방법
    - 필드에 람다 저장 후 재사용 및 테스트를 가능하게 함
    - 람다를 사용하는 메서드를 테스트
    - 람다식이 복잡할 경우 개별 메서드로 분할
    - 람다식의 결과가 함수를 반환할 경우 함수의 동작 자체 테스트

<h3>람다 테스팅</h3>

- `스택 트레이스`
    - 람다는 익명이기 때문에 스택 트레이스에 람다식의 이름이 표시되지 않음
    - 메서드 참조를 사용할 경우에도 메서드명이 표시되지않음
    - **메서드 참조를 사용하는 클래스와 같은 곳에 선언되어 있는 메서드를 참조 -> 해당 메서드 이름이 스택 트레이스에 나타남**
<br>

- `로깅`
    - Stream API의 `peek()`:  스트림의 각 요소를 소비한것처럼 동작

    ```java
            List<Integer> result = Stream.of(2, 3, 4, 5)
								// peek을 통해 map으로 전달될 요소들을 출력
                .peek(x -> System.out.println("taking from stream: " + x))
                .map(x -> x + 17)
                .peek(x -> System.out.println("after map: " + x))
                .filter(x -> x % 2 == 0)
                .peek(x -> System.out.println("after filter: " + x))
                .limit(3)
                .peek(x -> System.out.println("after limit: " + x))
                .collect(toList());
    ```

<h2>3. 람다를 이용한 도메인 전용 언어</h2>

<h3>DSL</h3>

- 프로그램은 사람이 이애할 수 있도록 작성되어야 하고 작성하는 의도가 명확하게 전달되어야 함
- `DSL`(Domain-Specific Languages)는 소프트웨어 영역에서 읽기 쉽고 이해하기 쉬운 코드를 만들기 위해 고안
- 장점
    - **간결함**<br>
    비지니스 로직을 캡슐화 -> 반복↓, 코드 간결
    - **가독성**<br>
    도메인 전문가 및 비 전문가도 코드를 쉽게 이해할 수 있음
    - **유지보수**<br>
    잘 설계된 DSL은 쉽게 유지보수 할 수 있고 바꿀 수 있음
    - **높은 수준의 추상화**<br>
    DSL은 도메인과 같은 추상화 수준에서 동작함 -> 도메인 비관련 세부 사항 숨김
    - **집중**<br>
    비즈니스 도메인의 규칙을 표현할 목적으로 설계됐으므로 프로그래머가 특정 코드에 집중할 수 있음
    - **관심사 분리**<br>
    비즈니스 로직을 지정된 언어로 표현하므로 애플리케이션의 인프라스트럭처와 쉽게 분리 가능 -> 유지보수가 쉬운 코드 생성
- 단점
    - **DSL 설계의 어려움**<br>
    제한적인 언어에 도메인 지식을 담는 것은 쉬운 작업이 아님
    - **개발 비용**<br>
    DSL을 추가하게 되므로 초기 프로젝트 및 DSL 유지보수에 많은 비용 소모됨
    - **추가 우회 계층**<br>
    DSL은 추가적인 계층으로 도메인을 감쌈 -> 비용
    - **새로 배워야 하는 언어**<br>
     DSL을 기존언어와 다른 언어로 쓴다면 새로운 언어를 배워야 함
    - **호스팅 언어 한계**<br>
     Java의 장황하고 엄격한 문법 -> 사용자 친화적 DSL을 만들기 어려움

<br>
<h3>JVM에서 이용할 수 있는 다른 DSL 해결책</h3>


<h4>내부 DSL</h4>

- 애플리케이션 수준 기본 요소를 데이터베이스를 나타내는 하나 이상의 클래스 유형에서 사용할 수 있는 Java 메서드로 노출
- `Java 언어로 구성된 DSL`
- 자바 언어의 부족한 유연성을 람다가 보충

```java
public static void main(String[] args) {
    List<String> numbers = Arrays.asList("one", "two", "three");
		
		// 1.
    System.out.println("Anonymous class:");
    numbers.forEach(new Consumer<String>() {

        @Override
        public void accept(String s) {
            System.out.println(s);
        }

    });
		
		// 2.
    System.out.println("Lambda expression:");
    numbers.forEach(s -> System.out.println(s));

		// 3.
    System.out.println("Method reference:");
    numbers.forEach(System.out::println);
}
```


- 1에 비해 2, 3 방식은 딱 필요한 코드만 존재
- 자바를 이용한 DSL의 장점

    - 새로운 패턴 기술을 배우는 것보다 노력이 현저히 줄어듦
    - 같은 자바 코드이므로 나머지 코드와 함께 DSL 코드를 컴파일 할 수 있음
    - 개발 팀이 새로운 언어를 배우거나 복잡한 외부 도구를 사용하지 않아도 됨
    - 기존 Java IDE 사용 가능
    - 한 개의 DSL 언어로 여러개의 도메인을 대응하지 못해 추가 DSL을 개발할때 쉽게 추가할 수 있음
<br>
<h4>다중 DSL</h4>

- JVM 기반에서 돌아가는 언어들을 활용한 DSL
- 좋은 DSL을 만들려면 고급 기능을 활용할 수 있는 충분한 지식 필요
- 두 개 이상의 언어가 혼재하므로 여러 컴파일러를 사용하여 소스를 빌드함
- JVM 실행 언어가 Java와 호환되지 않을 수 있음

<h4>외부 DSL</h4>

- 자신만의 문법과 구문으로 새로운 언어를 설계해야 함
- 새로운 언어를 만들기 때문에 많은 비용 필요
- 내 입맛대로 설계하면 되므로 무한한 유연성
- 자바로 개발된 인프라와 외부 DSL로 구현한 비즈니스 코드를 쉽게 분리

<br>

<h3>최신 자바 API의 작은 DSL</h3>

- `Comparator 인터페이스`는 람다 표현식을 활용해 정렬할 객체로 람다 활용 가능
```java
public class DslSortingWithComparator {
    public static void main(String[] args) {
        
        // Java 8 이전의 방식 직접 클래스를 구현함 -> 부가적인 코드가 너무 많음
        Collections.sort(menu, new Comparator<Dish>() {
            @Override
            public int compare(final Dish o1, final Dish o2) {
                return o1.getCalories() - o2.getCalories();
            }
        });
        
        // Java 8의 람다 사용
        Collections.sort(menu, Comparator.comparing(d -> d.getCalories()));
        // Java 8의 메서드 참조 사용
        Collections.sort(menu, Comparator.comparing(Dish::getCalories));
        // List 인터페이스의 sort 메서드 사용
        menu.sort(Comparator.comparing(Dish::getCalories));
    }
}
```

- `Stream API`의 fluent 형식은 잘 설계된 DSL
    - 중간연산은 지연동작되고 체인을 구성할 수 있게 스트림 자신을 반환
    - 최종 연산에서 모든 스트림 요소들을 사용하여 연산

<br>

<h3>데이터를 수집하는 DSL인 Collectors</h3>

- Stream API에서 최종 연산으로 `collect`를 사용하면 스트림의 요소들을 수집, 그룹화, 분할과 같은 작업을 수행 가능
- Collectors가 제공하는 여러 정적 팩토리 메서드를 활용해 필요한 Collector 객체를 만들고 합칠 수 있음

```java
// Comparator 메서드, fluent style
Comparator<Person> comparator =
        comparing(Person::getAge).thenComparing(Person::getName);

// Collectors 정적 팩터리 메서드, 중첩 메서드 스타일
Map<String, Map<Color, List<Car>>> carsByBrandAndColor = cars.stream()
				.collect(groupingBy(Car::getBrand, groupingBy(Car::getColor)));
```

<h3>자바로 DSL 만들기</h3>

<h4>메서드 체인</h4>

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

- 빌더 패턴으로 구성
- 필요에 따라 여러 빌터 클래스를 조합해 사용자에게 지정된 순서로 fluent api를 호출하도록 강제가능

- 장점
    - 메서드 이름 == 키워드 인수
    - 도메인 필드에서 선택형 데이터가 있을 경우 사용하기 적합
    - DSL 사용자에게 정해진 순서대로 메서드를 호출하도록 강제
    - 정적 메서드를 최소한으로 사용함
    - 의미없는 코드량이 적음

- 단점
 - 구현 코드량 많음
 - 사용할 빌더들을 연결하는 접착 코드 필요
 - 도메인의 객체의 중첩 구조와 일치하
게 들여쓰기를 강제하는 방법이 없음
<br>

<h4>중첩함수 활용</h4>

```java
Order order = order("BigBank",
        buy(80,
                stock("IBM", on("NYSE")),
                at(125.00)),
        sell(50,
                stock("GOOGLE", on("NASDAQ")),
                at(375.00))
);
```

- 장점
    - 구현의 장황함을 줄일 수 있음
    - 함수 중첩을 통해 도메인 객체 계층 표현 가능

- 단점
    - 정적 메서드의 사용이 많음
    - 이름이 아닌 위치를 통해 인수 정의
    - 도메인에 선택적인 필드가 있을 경우 별도의 메서드 오버라이드 필요
<br>

<h4>람다 표현식을 이용한 함수 시퀀싱</h4>

```java
Order order = LambdaOrderBuilder.order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
        t.quantity(80);
        t.price(125.00);
        t.stock(s -> {
            s.symbol("IBM");
            s.market("NYSE");
        });
    });
    o.sell(t -> {
        t.quantity(50);
        t.price(375.00);
        t.stock(s -> {
            s.symbol("GOOGLE");
            s.market("NASDAQ");
        });
    });
});
```

- Consumer를 받음 -> 사용자가 람다 표현식을 통해 인수 구현 가능
- 메서드 체인의 fluent style + 중첩함수 형식의 람다 적용 -> 도메인 객체의 계층 구조를 유지

- 장점
    - 도메인에 선택적인 필드가 있을 경우 적합
    - 정적 메서드 최소화
    - 람다 중첩을 통해 도메인 계층 반영
    - 빌더에 필요한 접착코드 필요X

- 단점
    - 구현 복잡
    - 람다 표현식으로 인해 필요없는 코드량↑

<h4>조합하여 사용</h4>

```java
Order order = forCustomer("BigBank",
        buy(t -> t.quantity(80)
                .stock("IBM")
                .on("NYSE")
                .at(125.00)),
        sell(t -> t.quantity(50)
                .stock("GOOGLE")
                .on("NASDAQ")
                .at(375.00)));
```

- 여러 DSL기법을 혼용하여 각 기법의 장점 조합

- 장점
    - 여러 기법중에서 필요한 기법 활용 -> 도메인 표현 최적화
- 단점
    - DSL 사용자가 조합한 여러 기법을 익히는데 시간비용↑
<br>

<h3>실생활의 자바8 DSL예제</h3>

<h4>JOOQ</h4>

- QueryDSL과 함께 자주 거론되는 자바 코드 기반 DB 쿼리 작성 가능
- 메서드 체인 패턴을 중점적으로 활용
- 필수적으로 입력되어야 하는 구문 -> 사용자에게 정해진 `순서대로 사용할 수 있게 강제`하거나 `선택적인 파라미터를 입력`하는데 적합

<h4>Cucumber</h4>

```java
public class BuyStocksSteps {

    private Map<String, Integer> stockUnitPrices = new HashMap<>();
    private Order order = new Order();

    @Given("^the price of a \"(.*?)\" stock is (\\d+)\\$$")
    public void setUnitPrice(String stockName, int unitPrice) {
        stockUnitValues.put(stockName, unitPrice);
    }

    @When("^I buy (\\d+) \"(.*?)\"$")
    public void buyStocks(int quantity, String stockName) {Trade trade = new Trade();
				trade.setType(Trade.Type.BUY);
				Stock stock = new Stock();
				stock.setSymbol(stockName);
				Populates the domain model accordinglytrade.setStock(stock);
		    trade.setPrice(stockUnitPrices.get(stockName));
		    trade.setQuantity(quantity);
		    order.addTrade(trade);
		}

@Then("^the order value should be (\\d+)\\$$")
    public void checkOrderValue(int expectedValue) {
        assertEquals(expectedValue, order.getValue());
    }
}
```


- BDD 지원용 툴로 외부 DSL을 활용하여 테스팅 가능
- 애노테이션을 통해 매칭하여 BDD 테스팅을 진행
- 람다 사용 -> 의미있는 메서드의 이름 필요X

<h4>Spring Integration</h4>

```java
@Configuration
@EnableIntegration
public class MyConfiguration {

    @Bean
    public MessageSource<?> integerMessageSource() {
        MethodInvokingMessageSource source =
                new MethodInvokingMessageSource();
        source.setObject(new AtomicInteger());
        source.setMethodName("getAndIncrement");
        return source;
		}

		@Bean
		public DirectChannel inputChannel() {
		    return new DirectChannel();
		}

		@Bean
		public IntegrationFlow myFlow() {
		    return IntegrationFlows.from(this.integerMessageSource(),c -> c.poller(Pollers.fixedRate(10)))
								.channel(this.inputChannel())
								.filter((Integer p) -> p % 2 == 0)
								.transform(Object::toString)
								.channel(MessageChannels.queue("queueChannel"))
								.get();
		}
}
```

- 메서드 체이닝을 주로 사용
- 필요할 경우 람다 표현식을 적용

<h3>마치며</h3>

- **DSL의 목적은 개발자와 도메인 전문가 사이의 간극을 좁히는 것**
    - 개발자가 아닌 사람도 이해할 수 있는 언어로 비즈니스 로직을 구현하기 위함
- DSL은 `내부 DSL`(자바 언어 혹은 해당 애플리케이션을 만들때 적용한 언어), `외부 DSL`(애플리케이션 언어와 달리 특화된 언어 혹은 직접 만든 언어) 크게 존재
- 다중 DSL을 개발할 수 있음(JVM 위에서 돌아가는 언어를 통해) → 통합시 문제 발생 여지 있음
- Java는 내부 DSL을 구현하기 적합치 않으나 람다 및 메서드 참조를 통해 많이 개선됨
- 최신 자바는 자체 API에서 DSL을 제공해주고 있음(Stream, Collectors, Comparator등)
- 자바의 DSL 패턴은 `메서드 체인, 중첩 함수, 함수 시퀀싱`과 같이 크게 3가지가 존재
- 여러 자바 프레임워크는 이미 DSL을 적극 활용해 구현되어 있음
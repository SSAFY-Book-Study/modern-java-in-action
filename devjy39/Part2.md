# Part2

# 4장 스트림 소개

- 자바에서는 다수의 데이터를 다룰 때 필수적으로 컬렉션을 사용한다. 자바 애플리케이션에서는 필수적으로 사용하는 것이다.
- 스트림을 이용하면 선언형으로 컬렉션을 가독성있게 사용할 수 있을 뿐더러 간단하게 병렬처리를 할 수 있다.

## 장점

- 기존의 반복문과 If문 없이 선언형, 즉 메서드 체이닝으로 가독성있고 편하게 컬렉션을 다룰 수 있다.
- 동작 파라미터화(람다, 함수형 인터페이스)를 사용하여 변화에 유연한 코드를 작성할 수 있다.
- 복잡한 데이터 처리 파이프라인을 각종 메서드를 조립하면서 유연하게 만들 수 있다.
- 동시성 문제 걱정 없이 병렬처리를 할 수 있다.

## 스트림 정의

데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- 연속된 요소 : 특정 요소 형식으로 이뤄진 연속된 값 집합의 인터페이스를 제공한다.
- 소스 : 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. 순서는 유지된다.
- 데이터 처리 연산 : 데이터베이스와 비슷한 연산을 지원한다. 순차 또는 병렬로 실행할 수 있다. ⇒ filter, map, limit, collect 등
- 파이프라이닝 : 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
- 내부 반복 : 반복문을 사용하는 대신 내부 반복자 iterator를 지원한다.

## 컬렉션과 스트림의 차이

- 둘 다 순차적으로 값에 접근할 수 있는 기능이지만 데이터를 언제 계산하느냐의 큰 차이가 있다.
- 컬렉션은 자료구조가 포함하는 모든 값을 메모리에 저장한다. 즉 계산 후 컬렉션에 저장되어야 하고, 계산 과정에서 추가적인 메모리가 필요할 수도 있다.
- 반면 스트림은 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조다. 즉 사용자가 데이터를 요청할 때만 값을 계산한다.

- 컬렉션의 경우 모든 값을 계산할 때까지 기다린다.
- 스트림의 경우 필요할 때만 값을 계산하는 Lady Loading이 가능하다.
- 스트림은 또한 모든 요소를 한 번만 탐색할 수 있다. 즉, 탐색된 요소는 소비되고 해당 스트림에서 탐색한 요소를 다시 탐색할 수 없다.

### 내부반복 외부반복

- 컬렉션은 반복문을 통해서 외부 반복을 통해 요소를 순회한다.
- 스트림은 반복을 알아서 처리해주고 결과 스트림 값을 어딘가에 저장해주는 내부반복을 사용한다.

## 스트림의 연산

- 중간 연산
    - 서로 연결되어 파이프라인을 형성한다. 스트림을 반환 함.
- 최종 연산
    - 파이프라인을 실행한 다음에 닫는다. 스트림이 아닌 값을 반환한다.
- 각각의 중간연산들은 최종 연산이 실행 될 때 한 번에 처리되기 때문에 Lazy한 연산이 가능하다.

스트림이 중간 연산마다 결과값을 도출하고 넘겨주는 줄 알았는데 전혀 아니었다.

스트림은 모든 파이프라인을 한 번에 처리하고 필요하지 않은 연산은 하지도 않고 있었다..

```java
List<Apple> list = new ArrayList<>();
list.add(new Apple(10,"한국"));
list.add(new Apple(20,"중국"));
list.add(new Apple(30,"미국"));
list.add(new Apple(40,"캐나다"));
list.add(new Apple(50, "일본"));

List<String> collect = list.stream()
        .filter(a -> {
            System.out.println("filtering : " + a);
            return a.getWeight() >= 20;})
        .map(a -> {
            System.out.println("mapping :" + a);
            return a.getCountry();
        })
        .limit(2)
        .collect(toList());

System.out.println(collect);

// output
filtering : Apple{weight=10, country='한국'}
filtering : Apple{weight=20, country='중국'}
mapping :Apple{weight=20, country='중국'}
filtering : Apple{weight=30, country='미국'}
mapping :Apple{weight=30, country='미국'}
[중국, 미국]
```

---

# 5장. 스트림 활용

### filter(Predicate)

- boolean 값에 의한 필터링

### distinct()

- 중복을 제거한 스트림을 반환한다.
- 중복 결정 여부는 hashCode, equals로 결정된다.

## 스트림 슬라이싱

- 정렬된 데이터의 스트림인 경우 요소의 일부를 효과적으로 선택하거나 버릴 수 있다.
    - 슬라이싱 기능들은 무한 스트림에서도 동작한다.
    - 정렬된 스트림일 경우 유용하다.
- JDK 9 기능
    - takeWhile (predicate) →  boolean 값이 false가 나온 순간 이후 요소들을 버린다.
    - dropWhile (predicate) → boolean값이 false가 나오기 전까지 요소를 모두 버린다.

### limit

- 스트림의 요소를 축소시킨다. 지정한 요소의 개수 만큼만 연산한다.

### skip

- 첫 n개의 요소를 건너 뛴 후(버림)의 스트림을 반환한다.
- 요소가 n개 이하라면 빈 스트림이 반환됨.

## 매핑

- map : 함수를 인수로 받아 함수의 반환값으로 새로운 요소로 변환한다.
- flatMap : 스트림의 각각의 값을 다른 스트림으로 만든 후 모든 스트림을 하나의 스트림으로 연결한다.
    - 반환값은 스트림이다.

- 2개의 스트림을 혼합해서 i + j 의 합이 3의 배수인 경우만 배열로 반환하기

```java
List<Integer> arr = List.of(1, 2, 3);
List<Integer> arr2 = List.of(3, 4);

arr.stream()
        .flatMap(i -> arr2.stream()
                .filter(j -> (i + j) % 3 == 0)
                .map(j -> new int[]{i, j}))
        .forEach(ar -> System.out.println(ar[0] + " " + ar[1]));
```

### 검색과 매칭

- 쇼트 서킷 기법, 자바의 &&, || 와 같은 연산을 사용하는 기능
- anyMatch(predicate) → 어느 하나라도 일치하는지 boolean 반환
- allMatch(predicate) → 모든 요소가 일치하는지 boolean 반환
- noneMatch(predicate) → 일치하는 요소가 없으면 true 반환

### 쇼트서킷

- 전체의 데이터를 보지 않고도 불리언 표현식을 평가할 수 있는 경우 연산을 중단하고 결과를 반환한다.
- limit도 쇼트서킷의 일종이다.

### findAny

- 일치하는 요소가 하나라도 발견되는 즉시 연산을 중단하고 반환한다.
- 일치하는 값이 없을 수도 있으므로 Optional을 반환한다.

### findFirst

- 일치하는 첫번째 요소를 반환한다.

### findAny vs findFirst

- 병렬연산의 경우 첫 번째 요소를 찾기 어렵다. 요소의 반환 순서가 상관없다면 동시성에서 제약이 적은 findAny를 사용한다.

## 리듀싱

- 모든 스트림 요소를 처리해서 값으로 도출하는 연산. ( 함수형 프로그래밍에선 폴드라고 부른다)
- reduce(0, (a,b) → a+ b) 의 경우 초기 a값 0부터 모든 요소를 하나의 값으로 줄어들 때 까지 반복해서 조합한다.
- 파라미터에 초기값을 넣지 않을 경우 Optional을 반환한다.
- map과 reduce를 연결하는 기법을 맵 리듀스 패턴이라고 한다.

### 장점

- reduce를 이용하면 내부 반복이 추상화 되면서 내부 구현에서 병렬로 실행할 수 있게 된다.
- 기본적으로 합을 구하는 연산에서는 sum이라는 공유 변수가 생기므로 동기화 문제가 발생하는데 parallelStream으로 동작하면 그 대신 대가가 필요하다.
- 넘겨준 람다가 stateless 여야 하고 순서가 상관 없는 구조여야한다.

## stateless

- map, filter 등의 내부적인 가변 상태를 갖지 않은 연산이다.
- 하지만 reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. 하지만 그렇다 하더라도 내부 상태의 크기는 한정되어있다.
- 반면 sorted나 distinct같은 연산은 모든 요소를 정렬하거나 중복 제거 하기 위해 과거의 이력을 알고 있어야 하고, 모든 요소가 버퍼에 추가되어 있어야 한다.
- 무한 스트림의 경우 정렬이나 중복 제거를 하면 어떻게 될까?, 이러한 연산을 내부 상태를 갖는 연산이라고 한다.

## 스트림의 모든 요소는 참조형변수이다.

- 기본적으로 auth boxing, unboxing 비용이 나갈 것이다.
- 이걸 위해 기본형 특화 스트림 int, double, long 을 제공한다. max, min, average 등의 연산을 제공한다.
- boxed()를 통해서 객체 스트림으로 복원할 수 있다.

## 스트림 표현

- 숫자 범위로 스트림 표현 
IntStream.range(1, 100) : 1, 100 미포함 , IntStream.rangeClosed(1, 100) : 1, 100 포함
- Stream.of( 원소들 ) 로 stream 표현
- Stream.empty(); 빈 스트림 생성
- Stream.ofNullable( nullable value ) : null이 될 수 있는 객체를 스트림화
- Arrays.stream( array ) : 배열을 stream으로
- 파일로 스트림 → Files.lines는 stream을 반환한다.
- stream은 AutoCloseable 이므로 try with resource에 사용될 수 있다.

### 무한 스트림 표현

- 요청할 때마다 값을 생성하는 무한 스트림
- iterate ( 초기값, 람다 ) : 초기값과 람다로 무한한 스트림을 생성한다.
    - 초기값, 종료식, 람다 : 종료식을 넣어 false가 나오면 중단하도록 할 수 있다. (takeWhile과 같은 동작)
    - filter는 종료식이 아니므로 다르다.
- generate (supplier) : 값을 생성하는 무한 스트림
    - interate와 다르게 상태를 갖지 않으면서 값을 생성할 수 있다. ( 물론 억지로 구현하면 상태를 넣을 순 있지만, 좋지 않은 코드이다)

### Collector

- collect() 메서드에 인자로 넣어서 원하는 자료형으로 변형시킬 수 있다. 주로 List, Map, Set 등의 자료형이다.
- **리듀싱 연산**을 통해 데이터 저장 구조를 변환한다.
  
- 스트림의 요소를 하나의 값으로 리듀스하고 요약.
    - 요약 연산 : Collectors.summingInt 라는 요약 팩토리 메서드를 제공한다. 
    sum, max, min, average 등의 연산을 제공해준다.
    - 모든 연산 결과가 저장된 객체인 IntSummaryStatistics 객체를 반환하는 summarizingInt 메서드도 제공해준다.
- Collector.joining() : 객체에 toString을 호출해서 하나의 문자열로 만들어 반환한다.
StringBuilder를 이용해서 문자열을 하나로 만들어서 문자열 덧셈 오버 헤드를 걱정하지 않아도되고 문자열 사이에 문자열을 넣을 수도 있다.

- 범용 Collectors.reducing(BiFunction<T, T, T>) 메서드를 응용해서 웬만한 제공되는 메서드를 구현할 수 있고 유연성과 확장성을 얻을 수 있다.

- 요소 그룹화
    - groupingby를 이용해서 Map으로 그룹화 할 수 있다.
    - 람다식을 인수로 복잡한 기준을 넣어서 그룹화 할 수 있다.
    - mapping 함수를 넣어서 value를 지정할 수 있다.
    - flatMapping은 stream을 flat 해준다.
    - groupingBy 안에  groupingBy를 넣어서 Map안의 Map인 트리구조로 그룹화할 수도 있다.
    - groupingBy 컬렉터는 스트림의 첫 번째 요소를 찾은 후에야 새로운 키를 추가하기 때문에 Option을 value로 넣을 필요가 없다.
    collectingAndThen(maxBy(..), Optional::get))으로 Optional을 삭제할 수 있다.

- 요소 분할
    - 그룹화 키를 predicate(true, false)로 갖는 그룹화 기능이다.
    - partitioningBy(predicate)를 이용할 수 있으며, 장점은 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이다.
    - 그룹화처럼 Collector 메서드를 전달해서 value 값에 두 수준의 중첩 그룹화도 가능하다.

## Collector 인터페이스

- 리듀싱 연산에 사용될 Collector를 커스텀하여 만들 수 있게 인터페이스를 제공해준다.
- Collector < T (수집될 항목), A (중간 연결과를 누적하는 누적자 ), R (수집 연산 결과 객체)>

### supplier()

- 누적자 A, 중간 결과를 누적하기 위한 객체를 반환하는 supplier

### accumulator()

- 리듀싱 연산을 직접 수행하는 함수 , <누적자, 수집항목>

### finisher()

- 누적 과정이 끝날 때 누적자 객체를 최종 결과로 반환할 함수

### combiner()

- 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이결과를 어떻게 처리할지 정의한다.
- 누적자 끼리의 연산, 두 결과 컨테이너(서브스트림) 병합되며, 리듀싱 연산을 통해 결국 하나의 결과 요소로 반환되는 것이다.

### characteristics()

- 스트림을 병렬로 리듀스할 것인지, 그렇다면 어떤 최적화를 선택해야 할지 힌트를 제공한다.
- 다음과 같은 옵션들의 열거형이다.
    - UNORDERED : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 **순서에 영향을 받지 않는다.**
        - 순서 상관 없음
    - CONCURRENT : 다중 스레드에서 accumulator를 동시에 호출할 수 있다.
        - 병렬 처리 가능 ( stateless )
    - IDENTITY_FINISH : finisher 메서드를 생략될 수 있고, 최종 결과로 누적자 객체를 바로 사용할 수 있다.
        - 피니쉬 연산 필요 없음

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();
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
        return Collections.unmodifiableSet(EnumSet.of(
                IDENTITY_FINISH, CONCURRENT));
    }
}

// 발행자, 누적자, 합침자 만 인수로 넣고 커스텀 구현하기
stream.collect( ArrayList::new, List::add, list::addAll ) 
// 간결하지만 가독성은 떨어지고, Characteristics는 전달할 수 없다.
```

---

# 6장. 병렬 데이터 처리와 성능

- 컬렉션을 병렬로 처리하려면 어떻게 해야할까? 우선 데이터를 서브파트로 분할하고 각각의 스레드에 할당한 다음 레이스 컨디션이 발생하지 않도록 적절한 동기화를 수행하고 마지막으로 부분 결과를 합쳐야 한다.
    - 레이스 컨디션(race condition) : 여러 ****프로세스가 공유 자원을 병행적으로 읽고 쓸 때 접근 순서에 따라 실행 결과가 달라지는 상황
- 자바 7에서는 병렬화를 더 쉽게하고 에러를 최소화할 수 있도록 포크/조인 프레임워크를 제공한다.

## Parallel Stream

- 각 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 병렬 스트림을 말한다.
- parallelStream(), parallel() 을 호출하는 것으로 만들 수 있다.
- 반대로 sequential()로 병렬 스트림을 순차 스트림으로 바꿀 수 있다.
    - 두 함수는 불리언 플래그를 true, false로 바꾸는 것일 뿐 최종적으로 파이프라인의 마지막 호출에서의 불리언 값만 보게된다.

## 병렬 스트림에서 스레드 풀

- 내부적으로 ForkJoinPool을 사용한다.
- Runtime.getRuntime().availableProcessors()가 반환하는 값에 상응하는 스레드를 갖는다.
- System 전역 설정으로 설정이 가능하지만 모든 연산에 영향을 주므로 기본값이 권장된다.

## 주의사항

- parallel stream을 사용하면 항상 빠를까? 그렇지 않다. 많은 것을 고려해야한다.
- 연산이 서브 스트림으로 나뉠 수 있는지 결과를 합칠 수 있는지
- boxing unboxing 연산이 일어나진 않는지
- 멀티코어 간의 데이터 이동은 생각보다 비싸서 코어 간 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 수행하는 것이 바람직하다.

## 병렬 스트림 효과적인 사용법

- 순차 스트림과 병렬 스트림 중 어떤 것이 좋을지 모르겠다면, 직접 **측정하라**.
- 성능 저하 요소인 **오토박싱을 주의**하라, 기본형 특화 스트림을 사용해라.
- 순차 스트림보다 병렬 스트림에서 **성능이 떨어지는 연산들을 조심**해라.
    - limit, findFirst 같이 요소의 순서에 의존하는 연산들이다.
    - sorted 스트림에 unordered()를 호출하면 비 정렬 스트림을 얻을 수 있고 여기에 limit를 호출하는게 더 효율적이다.
- 스트림에서 수행하는 전체 **파이프라인 연산 비용을 고려**하라.
    - 하나의 요소를 처리하는 데 드는 비용이 클 수록 병렬 연산으로 개선 가능성이 크다.
- **소량의 데이터에서는 병렬 스트림을 쓰지 말자**. 병렬화 과정에서 생기는 부가 비용이 배보다 배꼽이 더 커지기 때문이다.
- 스트림을 구성하는 **자료구조를 확인**해라.
    - LinkedList는 분할하려면 모든 요소를 탐색해야 한다.
    - Stream.iterate 도 순차적으로 생성되므로 좋지 않다.
    - range 팩토리 메서드로 만든 기본형 스트림은 쉽게 분해할 수 있다.
- 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 분해 과정의 성능이 달라진다.
    - **filter 연산**의 경우 스트림의 길이를 예측할 수 없으므로 효과적인 병렬 처리를 할 수 있을지 알 수 없게된다.
- 최종 연산의 **병합과정 비용**을 살펴봐라.
    - 병합 과정의 비용이 비싸다면 합치는 과정에서 이익이 상쇄될 수 있다.

## 포크/조인 프레임워크

- 자바 7에서 추가된 프레임워크로 병렬 스트림을 처리한다.
- 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 서브태스트 결과를 합쳐서 전체 결과를 만들도록 설계되었다.
- 스레드 풀의 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.

### RecursiveTask

- 재귀적으로 작업을 분할하는 메서드, 분할 정복 알고리즘의 병렬화 버전이다.
1. 태스크가 더 이상 분할할 수 없으면 태스트 계산.
2. 아니면 태스크를 두 서브태스크로 분할
3. 모든 서브태스트의 연산이 완료되면 결과를 합침

### 제대로된 사용법

- Task의 join() 메서드를 호출하면 결과를 받을 때까지 호출자를 블록시킨다.
    - 따라서, 두 서브태스크가 모두 시작된 다음 호출해야 한다.
- compute는 동기실행, fork는 비동기 실행, 한쪽 작업은 동기실행을 해서 호출 스레드를 재사용하여 불필요한 태스트 할당 오버헤드를 피하자.
- 병렬 계산은 디버깅하기 어려우므로 스택 트레이스는 도움이 되지 않는다.
- 각 서브태스크의 실행시간은 새로운 태스크를 포킹하는데 드는 시간보다 길어야 한다.
- 컴파일러 최적화는 병렬보다 순차 버전에 집중될 수 있다. → 죽은 코드를 분석해서 사용되지 않는 계산은 아예 삭제하는 등의 최적화를 달성하기 쉬움

## 작업 훔치기 work stealing

- 각각의 서브태스크의 작업완료 시간이 크게 달라져서 협력하는 과정에서 지연이 생길 수 있는데, 작업 훔치기 기법을 사용하여 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다.
- 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와서 작업을 처리한다.
- 일을 먼저 끝낸 스레드는 다른 스레드 큐의 꼬리에서 작업을 훔쳐오고 모든 큐가 빌때까지 이 과정을 반복한다.
- 따라서 태스크의 크기를 작게 나눠야 작업자 스레드 간의 작업 부하를 비슷하게 유지할 수 있다.

## Spliterator

- 분할할 수 있는 반복자 인터페이스
- iterator 처럼 탐색 기능을 제공하지만 병렬 작업에 특화되어있고, 스트림을 어떻게 병렬화할 것인지 정의한다.
```java
public interface Spliterator<T> {
    // 탐색해야 할 요소가 남아있으면 true 반환
		boolean tryAdvance(Consumer<? super T> action);
		
		// 요소를 분할하여 두 번째 Spliterator 생성
    Spliterator<T> trySplit();
		
		// 탐색해야 할 요소 수 정보
    long estimateSize();
		// Spliterator 자체의 특성 집합을 포함하는 int를 반환한다., 특성을 참고해서 각각을 더 잘 제어하고 최적화 한다.
    int characteristics();
}
```

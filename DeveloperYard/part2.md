# 함수형 데이터 처리

## Chapter4 - 스트림 소개

> 거의 모든 자바 애플리케이션은 컬렉션을 만들고 처리하는 과정을 포함한다. 컬렉션으로 데이터를 그룹화하고 처리할 수 있다.

<br>

### 스트림이란 무엇인가?

> 스트림은 Java 8 API에 새로이 추가된 기능이다. 스트림을 이용하면 선언형(즉, 데이터를 처리하는 임시 구현 코드 대신 질의로 표현할 수 있다.)

다음은 기존 코드이다.
```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu){
    if(dish.getCalories() < 400){
        lowCaloricDishes.add(dish);
    }    
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>(){
    public int compare(Dish d1, Dish d2){
        return Integer.compare(d1.getCalories(), d2.getCalories());            
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes){
    lowCaloricDishesName.add(dish.getName());
}
```

다음은 업데이트된 최신 코드이다.

```java
List<String> lowCaloricDishesName = 
    menu.stream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
```
Java 8의 스트림 API의 특징을 다음처럼 요약할 수 있다.
- 선언형 : 더 간결하고 가독성이 좋아진다.
- 조립할 수 있음 : 유연성이 좋아진다.
- 병렬화 : 성능이 좋아진다.

### 스트림 시작하기

> 스트림이란 '데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소'로 정의할 수 있다.

- 연속된 요소 : 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다.
- 소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
- 데이터 처리 연산 : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다.

또한 스트림에는 다음과 같은 두 가지 중요한 특징이 있다.

- 파이프라이닝 : 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
- 내부 반복 : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다.

다음 코드를 통해 알아보자.

```java
import static java.util.stream.Collector.toList;

List<String> threeHighCaloricDishNames = 
    menu.streeam()
        .filter(dish -> dish.getCalories() > 300)
        .map(Dish::getName) // extract dish name
        .limit(3) // pick first 3 element
        .collect(toList()); // store result to different result

System.out.println(threeHighCaloricDishNames);
```

### 스트림과 컬렉션

> 자바의 기존 컬렉션과 새로운 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 여기서 '연속된'이라는 표현은 순서와 상관없이 아무 값에나 접속하는 것이 아닌 순차적으로 값에 접근한다는 것을 의미한다.

### 외부 반복과 내부 반복

> 컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다. 이를 외부 반복이라 한다. 반면 스트림 라이브러리는 내부 반복을 사용한다.

```java
// external iteration
List<String> names = new ArrayList<>();
for(Dish dish : menu){
    names.add(dish.getName());    
}

// internal iteration
List<String> names = menu.stream()
        .map(Dish::getName())
        .collect(toList());
```

그렇다면 왜 내부 반복과 외부 반복을 나눌까?
스트림 라이브로리 내부 반복은 데이터 표현과 하드웨어를 활용한 **병렬성 구현을 자동으로 선택**한다.

반면, for-each를 이용하는 외부 반복에서는 병렬성을 **스스로 관리**해야 한다.

### 스트림 연산

> 연결할 수 있는 스트림 연산을 **중간 연산**이라고 하며, 스트림을 닫는 연산을 **최종 연산**이라고 한다.

#### 중간 연산

filter나 sorted같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다.

#### 최종 연산

최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다.

## chapter5 - 스트림 활용

### 필터링

- 프레디케이트로 필터링
> 스트림 인터페이스는 filter 메서드를 지원한다. filter 메서드는 프레디케이트를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

```java
// filtering example
List<Dish> vegetarianMenu = menu.stream()
        .filter(Dish::isVegeterian)
        .collect(toList())
```

- 고유 요소 필터링
> 스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다. 예를 들어 다음 코드는 리스트의 모든 짝수를 선택하고 중복을 필터링한다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
        .filter(i -> i % 2 == 0)
        .distinct()
        .forEach(System.out::println);
```

### 스트림 슬라이싱

- TAKEWHILE 활용

> takeWhile을 이용하면 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스할 수 있다.

- DROPWHILE 활용

> dropWhile은 takeWhile과는 반대의 작업을 수행한다. dropWhile은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다. 프레디케이트가 거짓이 되면 그 지점에서 작업을 중단하고 남은 모든 요소를 반환한다.

### 매핑

> 특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다. 스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

### 리듀싱

> 스트림 요소를 조합해 더 복잡한 질의를 표현하는 방법이다. 이러한 질의를 수행하려면 Integer같은 결과가 나올 때까지 스트림의 모든 요소를 반복적으로 처리해야 한다. 이런 질의를 **리듀싱 연산**(모든 스트림 요소를 처리해서 값으로 도출하는)이라고 한다.

```java
// 합을 구하는 반복
// for-each
int sum = 0;
for(int x : numbers){
    sum += x;
}

// use reducing
int sum = numbers.stream().reduce(0, (a, b)->a+b);
// 초기값 0
// 두 요소를 조합하여 새로운 값을 만드는 BinaryOperator<T>
```

### 숫자형 스트림

다음 예제를 통해 스트림 요소의 합을 구하는 코드를 알아보자.

```java
int calories = menu.stream()
        .map(Dish::getCalories)
        .reduce(0, Integer::sum);
```

하지만, 이렇게 계산하기 위해서는 내부적으로 합계를 계산을 하기 전에 Integer를 기본형으로 언박싱해야 한다.

이러한 박싱 비용을 줄이기 위해 스트림 API에서는 **기본형 특화 스트림**을 제공한다.

### 기본형 특화 스트림

#### 숫자 스트림으로 매핑

```java
int calories = menu,stream()
        .mapToInt(Dish::getCalories)
        .sum();
```

mapToInt 메서드는 각 요리에서 모든 칼로리(Integer format)를 추출한 다음에 Stream<Integer>가 아닌 IntStream을 반환한다. 따라서 IntStream 인터페이스에서 제공하는 sum 메서드를 이용해서 칼로리 합계를 계산할 수 있다.

#### 객체 스트림으로 복원하기

숫자 스트림을 생성 후, 특화되지 않은 스트림으로 다음과 같이 변환할 수 있다.

```java
IntStream intStream = menu.stream()
        .mapToInt(Dish::getCalories);

Stream<Integer> stream = intStream.boxed();
```

## chapter6 - 스트림으로 데이터 수집

> 중간 연산은 한 연 스트림을 다른 스트림을 변환하는 연산으로서, 여러 연산들을 연결할 수 있다. 이는 스트림 파이프라인을 구성하며, 스트림의 요소를 소비하지 않는다. 반면 최종 연산은 스트림의 요소를 소비해서 최종 결과를 도출한다. 최종 연산은 스트림 파이프라인을 최적화하면서 계산 과정을 짧게 생략하기도 한다.

### 컬렉터란 무엇인가?

#### 고급 리듀싱 기능을 수행하는 컬렉터

> 훌륭하게 설계된 함수형 API의 또 다른 장점으로 높은 수준의 조합성과 재사용성을 꼽을 수 있다. collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다.

#### 미리 정의된 컬렉터

Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

### 리듀싱과 요약

- collect와 reduce

> collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면 reduce는 두 값을 하나로 도출하는 불변형 연산이라는 점에서 의미론적인 문제가 발생한다. 가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 collect 메서드로 리듀싱 연산을 구현하는 것이 바람직하다.

### 그룹화

> 데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행되는 작업이다. 다음 예제를 살펴보자.

```java
Map<Dish.Type, List<Dish>> dishesByType = 
    menu.stream().collect(groupingBy(Dish::getType));
```

스트림의 각 요리에서 Dish, Type과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달했다. 이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류 함수**라고 한다.

### 분할

> 분할은 분할함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다. 분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 두 개의 그룹으로 분리된다.

```java
Map<Boolean, List<Dish>> partitionedMenu = 
    menu.stream().collect(partitioningBy(Dish::isVegiterian));
```

#### 분할의 장점

분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.

### Collector 인터페이스

> Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

#### Collector 인터페이스의 메서드 살펴보기

- supplier 메서드 : 새로운 결과 컨테이너 만들기
- accumulator 메서드 : 결과 컨테이너에 요소 추가하기
- finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기
- combiner 메서드 : 두 결과 컨테이너 병합
- Characteristic 메서드
> characteristic 메서드는 컬렉터의 연산을 정의하는 Characteristic 형식의 불변 집합을 반환한다. Characteristic는 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공한다.

#### 커스텀 컬렉터를 구현해서 성능 개선하기

```java
// n 이하의 자연수를 소수와 비소수로 분류하기
public Map<Boolean, List<Integer>> partitionPrimes(int n){
    return IntStream.rangeClosed(2, n).boxed()
        .collect(partitioningBy(candidate -> isPrime(candidate)));    
}
```

그리고 제곱근 이하로 대상의 숫자 범위를 제한해서 isPrime 메서드를 개선한다.

```java
public boolean isPrime(int candidate){
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
        .noneMatch(i -> candidate % i == 0);
}
```

여기서 성능을 더 개선할 수 있을까? 커스텀 컬렉터를 개선하면 성능을 더 개선할 수 있다.

#### 소수로만 나누기

중간 결과 리스트가 있다면 isPrime 메서드로 중간 결과 리스트를 전달하도록 다음과 같이 구현할 수 있다.

```java
public static boolean isPrime(List<Integer> primes, int candidate){
    return primes.stream().nonMatch(i -> candidate % i == 0);    
}

// use takeWhile method
public static boolean isPrime(List<Integer> primes, int candidate){
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return primes.stream()
        .takeWhile(i -> i <= candidateRoot)
        .noneMatch(i -> candidate % i == 0);
}
```

## chapter7 - 병렬 데이터 처리와 성능

> 자바 7이 등장하기 전에는 데이터 컬렉션을 병렬로 처리하기가 어려웠다. 우선 데이터를 서브파트로 분할해야 한다. 그리고 분할된 서브파트를 각각의 스레드로 할당한다. 스레드로 할당한 다음에는 의도치않은 레이스 컨디션이 발생하지 않도록 적절한 동기화를 추가해야 하며, 마지막으로 부분 결과를 합쳐야 한다.

### 병렬 스트림

> 컬렉션에 parallelStream을 호출하면 병렬 스트림이 생성된다. 병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 청크로 분할한 스트림이다.

숫자 n을 인수로 받아 1부터 n까지의 모든 숫자의 합계를 반환하는 메서드를 구현한다고 가정해보자.

```java
public long sequentialSum(long n){
    return Stream.iterate(1L, i -> i + 1)
        .limit(n)
        .reduce(0L, Long::sum);
}

// 기존 자바
public long iterativeSum(long n){
    long result = 0;
    for(long i = 1L; i <= n; i++){
        result += i;    
    }
    return result;
}
```

순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산(숫자 합계 계산)이 병렬로 처리된다.

```java
public long parallelSum(long n){
    return Stream.iterate(1L, i -> i + 1)
        .limit(n)
        .parallel()
        .reduce(0L, Long::sum);
}
```

다음 코드는 리듀싱 연산을 여러 청크에 병렬로 수행할 수 있고, 마지막으로 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐서 전체 스트림의 리듀싱 결과를 도출한다.

#### 병렬 스트림의 올바른 사용법

> 병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어난다. 다음의 코드는 n까지의 자연수를 더하면서 공유된 누적자를 바꾸는 프로그램을 구현한 코드다.

```java
public long sideEffectSum(long i){
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total = 0;
    public void add(long value) total += value;
}
```

위 코드는 본질적으로 순차 실행할 수 있도록 구현되어 있으므로 병렬로 실행하면 참사가 일어난다. 특히 total을 접근할 때마다 (다수의 스레드에서 동시에 데이터에 접근하는)데이터 레이스 문제가 일어난다. 동기화로 문제를 해결하다보면 결국 병렬화라는 특성이 없어져 버릴 것이다. 스트림을 병렬로 만들어서 어떤 문제가 일어나는지 확인하자.

```java
public long sideEffectParallelSum(long n){
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n)
        .parallel().forEach(accumulator::add);
    return accumulator.total;
}
```
메서드의 성능은 둘째 치고, 제대로 된 결과값이 나오지 않는다. 이는 여러 스레드에서 동시에 누적자, 즉 total += value를 실행하기 때문에 이러한 문제가 발생한다.

### 포크 / 조인 프레임워크

> 포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음에 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계되었다. 포크/조인 프레임워크에서는 서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExecutorService 인터페이스를 구현한다.

- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리할 수 있다.
- 간단하게 스트림을 병렬로 처리할 수 있지만 항상 병렬 처리가 빠른 것은 아니다. 병렬 소프트웨어 동작 방법과 성능은 직관적이지 않을 때가 많으므로 병렬 처리를 사용했을 때 성능을 직접 측정해봐야 한다.
- 병렬 스트림으로 데이터 집합을 병렬 실행할 때 특히 처리해야 할 데이터가 아주 많거나 각 요소를 처라하는데 오랜 시간이 걸릴 때 성능을 높일 수 있다.
- 가능하면 기본형 특화 스트림을 사용하는 등 올바른 자료구조 선택이 어떤 연산을 병렬로 처리하는 것보다 성능적으로 더 큰 영향을 미칠 수 있다.
- 포크/조인 프레임워크에서는 병렬화할 수 있는 태스크를 작은 태스크로 분할한 다음에 분할된 태스크를 각각의 스레드로 실행하며 서브태스크 각각의 결과를 합쳐서 최종 결과를 생산한다. 







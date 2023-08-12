# Part1

# 1장. 자바 8 ~ 11 변화의 흐름

### 자바가 변화하는 이유

- 언어는 하드웨어나 프로그래머 기대의 변화에 부응하지 않으면 도태되기 때문에 자바는 계속해서 새로운 기능을 추가하고 있다.

## 자바 8에서 추가된 대표적인 기능

### Stream API

- 컬렉션을 처리하면서 발생하는 모호함과 반복적인 보일러플레이트코드를 줄일 수 있다.

- stream을 이용하면 멀티쓰레드 사용으로 비용이 비싼 synchronized 를 사용하지 않고도 병렬연산을 안전하게 수행할 수 있다.
    - 어떻게 동기화 할까?
        - 큰 스트림을 작은 스트림으로 분할하여 연산한다.
- filter에서 병렬연산 예시
    1. 포킹단계 : 리스트를 반으로 쪼개서 각각 cpu에 요청
    2. 앞 부분과 뒷부분을 각각 연산 처리
    3. 연산 결과를 하나의  cpu가 정리
- 스트림은 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공하므로 
컬렉션의 가장 빠른 필터링 방법은 스트림으로 바꾸고 병렬처리 후 리스트로 다시 복원하는 것이 될 수 있다.

### Lambda, 메서드 참조

- 메서드를 일급 값로  만들 수 있다.
- 일급 값 ( 일급 객체, 시민 ) 이란?
    - 1. 변수나 데이타에 할당 할 수 있어야 한다.
        
        2. 객체의 인자로 넘길 수 있어야 한다.
        
        3. 객체의 리턴값으로 리턴 할수 있어야 한다.
        

- 람다 - 익명 함수 정의
    - 함수를 일급 값으로 만들 수 있다.
    - 한두 번만 사용할 메서드를 매번 정의할 필요 없이 람다로 표현할 수 있다.
    - 하지만 람다가 몇 줄 이상으로 길어진다면 코드의 명확성을 위해 코드의 수행 동작을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드참조를 사용하는게 바람직하다.

### Interface Default method

- 인터페이스를 업데이트하려면 해당 인터페이스의 구현체 모두 변경해야 하는데 이의 해결책으로 등장
- 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공해 주는 기능이다.

### Optional class

- nullPointer 를 피할 수 있는 기능

---


# 2장 동작 파라미터화 코드 전달하기

- 동작 파라미터화 : 아직 동작이 결정되지 않은 코드블록
    - 전략패턴을 통해 확장성 있고 재활용 가능한 구조
    - 변화하는 요구사항에 더 잘 대응할 수 있음, 결과적으로 엔지니어링 비용 감소

- 람다 이전에는 메서드를 클래스로 감싸서 전달해야했다.
보일러 플레이트가 너무 많음

- 람다를 사용해서 익명클래스를 만들지 않고 동작을 인수로 전달함으로써 간결성과 유연함이 증가된다.

---

# 3장 람다 표현식

- 람다는 전달할 수 있는 익명함수를 단순화한 것.
    - 특정 클래스에 종속되지 않으므로 메서드가 아닌 함수라고 표현

## 람다의 구조

- 파라미터 + → ( 화살표, 파라미터와 바디를 구분 ) + 바디

```java
// 표현식 스타일, return 명시 X
(Apple a1, Apple a2) -> Integer.compare(a1.getWeight(), a2.getWeight())

// 블록 스타일, return 있으면 명시
() -> { System.out.println(1); System.out.println(2); return 3; }

// void 한 줄은 중괄호 생략 가능
() -> System.out.println(1)
```

- 바디에는 return이 함축되어 있으므로 명시하지 않아도 됨.
- 바디에 여러행을 사용할 경우 중괄호 사용
- 한 개의 void 메서드 호출은 중괄호로 감쌀 필요 없음.

## 함수형 인터페이스

- 정확히 하나의 추상 메서드를 지정하는 인터페이스, 디폴트 메서드는 있어도 상관없음

함수형 인터페이스 이름 : 함수 디스크립터

Predicate<T>   : T → boolean

Consumer<T>  : T → void 

Supplier<T>    :  () → T

Comparator<T> : (T, T) → int

Runnable      :   () → 

Callable<V>  :  () → V

BiPredicate<T,U>  : (T,U) → boolean

IntBinaryOperator  : (int, int) → int

- 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있게된다.

- 기본 제공 이외에 함수형 인터페이스 만드는법 → @Functionalinterface 어노테이션을 사용하면 조건에 맞지 않을 시 컴파일러가 잡아준다.

```java
public static <T, R> List<R> map(List<T> list, Function<T, R> function) {
        List<R> result = new ArrayList<>();
        for (T t : list) {
            result.add(function.apply(t));
        }
        return result;
}

// 제네릭 T: String, R: int 가 자동으로 들어간다.
List<Integer> map = map(Arrays.asList("1", "12"), (String s) -> s.length());
```

- 자바에서 기본형 변수 오토박싱은 힙에 저장되며 메모리가 소비되고, 기본형으로 가져올 때도 메모리를 탐색하는 과정이 추가된다.
- 자바에서는 이러한 오토박싱을 피할 수 있도록 특별한 함수형 인터페이스를 제공한다

```java
Predicate<Integer> predicate = (Integer i) -> i == 0;
IntPredicate intPredicate = (int i) -> i == 0;

// 오토박싱
System.out.println(predicate.test(1));

// 박싱 없음
System.out.println(intPredicate.test(1));
```

- IntConsumer, LongFunction 이런 네이밍으로 제공해준다.

## 람다에서 예외

- 람다에서 예외를 던지려면 함수형 인터페이스를 만들고 예외를 명시하거나
- 블록 스타일 람다에서 try catch로 감싸야한다.

## 컴파일러에서 람다

- 람다식을 컴파일러가 마주했을 때 어떤 일이 일어날까?
    1. 제네릭이 특정 타입으로 대체되고 추상 메서드의 디스크립터를 확인.
    2. 함수 디스크립터와 람다식의 디스크립터가 일치하는지 확인한다.

```java
Object o = (Runnable) () -> System.out.println("hi");

// Action 함수형 인터페이스로 명시
Execute((Action) () -> {});
```

- 람다는 결국 함수형 인터페이스의 구현체 클래스이기 때문에 캐스팅하여 Object에 레퍼런스될 수 있다.
- 함수형 인터페이스는 함수 디스크립터만 같으면 어떤 인터페이스든 대체가 가능하기 때문에 캐스팅하여 명시해줄 수도 있다.

### 형식 추론

- 람다 표현식이 사용된 컨텍스트를 통해 파라미터 형식을 추론할 수 있으므로 람다 파라미터에 타입을 생략할 수 있다.
- 파라미터가 하나뿐일 때는 괄호도 생략 가능
- 상황에 따라 어떤 게 가독성이 좋은지 판단하여 사용하자.

### 지역변수 사용

- 람다캡쳐링 : 람다에서 외부 변수를 사용할 때는 final이거나 final처럼 사용되어야한다( 할당 x )

```java
int a = 5;
IntPredicate intPredicate = (i) -> a == 0;
a = 3; // 에러 발생
```

- 이유
    - 일반적으로 지역변수는 스택 메모리에 올라간다. 람다가 스레드에서 실행된다면 변수 할당이 해제되어 그 변수가 없어졌는데도 접근하려 할 수 있다.
    - 따라서 자바에서는 자유 지역 변수의 복사본을 제공하고, 복사본은 값이 바뀌지 않아야 하므로 이러한 제약이 생긴다.
    - 또한 병렬화 시  동시성 문제가 생길 수 있음.

## 클로저 (closure)

- 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스
- 클로저는 외부에 정의된 변수의 값에 접근 및 바꿀 수 있다.
- 람다와 익명 클래스는 비슷한 동작을 수행한다.
- 하지만 람다가 정의된 지역변수의 값은 바꿀 수 없다.


## 메서드 참조

- 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.
- 명시적으로 메서드명이 참조되므로 가독성이 증가한다.
- :: 구분자를 붙이는 방식으로 표현한다.
- 함수형 인터페이스에 대입할 때 파라미터와 반환 값은 선언된 함수와 일치해야 함.

```java
(String s) -> System.out.println(s) => System.out::println
(String str) -> str.toUpperCase() => String::toUpperCase

// 클래스 변수명으로도 참조 가능
Car car = new Car();
car::getFuel

// this로 참조 가능
this::method 

// 생성자는 new로 참조 가능
Car::new

// 이런 표현이 가능
Supplier<Apple> s = Apple::new;
Apple apple = s.get();

```

## 람다 표현식을 조합할 수 있는 유용한 디폴트 메서드

- 디폴트 메서드를 사용해서 람다를 더 유연하고 가독성있게 사용할 수 있다.

```java
List<Apple> list = new ArrayList<>();
list.add(new Apple(10, "한국"));

// 디폴트 메서드를 제공하는 Comparator
list.sort(Comparator.comparingInt(Apple::getWeight)
            .reversed() // 역순
            .thenComparing(Apple::getCountry)); // 같으면 비교


Apple apple1 = list.get(0);

// predicate에서 제공하는 디폴트 메서드 and, or, negate
Predicate<Apple> predicate = apple -> apple.weight == 0;
System.out.println(predicate.negate().test(apple1));
System.out.println(predicate.and(apple -> apple.country.equals("한국")).test(apple1));
System.out.println(predicate.or(apple -> apple.country.equals("한국")).test(apple1));

// Function 에서 andThen, compose
Function<Integer, Integer> f = x -> x + 10;
Function<Integer, Integer> g = x -> x * 10;
Function<Integer, Integer> andThen = f.andThen(g); // f -> g
Function<Integer, Integer> compose = f.compose(g); // g -> f

System.out.println(andThen.apply(1));
System.out.println(compose.apply(1));
```

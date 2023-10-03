# Part6 - 함수형 프로그래밍과 자바 진화의 미래

## Chapter18 - 함수형 관점으로 생각하기

&nbsp;

> 많은 프로그래머들이 유지보수 중 코드 크래시 디버깅 문제를 가장 많이 겪게 된다.

코드 크래시는 예상치 못한 변숫값 때문에 발생할 수 있다. 함수형 프로그래밍이 제공하는 **부작용 없음**과 **불변성**이라는 개념이 문제를 해결하는데 도움을 준다.

&nbsp;

### 공유된 가변 데이터

&nbsp;

> 리스트를 참조하는 여러 클래스들이 있다고 가정하자. 리스트의 소유자는 어느 클래스일까?

&nbsp;

어떤 자료구조도 바꾸지 않는 시스템이 있다고 가정하면 매우 유지보수하기 쉬워질 것이다.

자신을 포함하는 클래스의 상태 그리고 다른 객체의 상태를 바꾸지 않으며 return문을 통해서만 자신의 결과를 반환하는 메서드를 **순수**메서드 또는 **부작용 없는**메서드라고 부른다.

부작용에는 다음과 같은 것들이 있다.

- 자료구조를 고치거나 필드에 값을 할당
- 예외 발생
- 파일에 쓰기 등의 I/O 동작 수행

&nbsp;

### 선언형 프로그래밍

&nbsp;

> '어떻게'에 집중하는 프로그래밍 형식은 고전의 객체지향 프로그래밍에서 이용하는 방식이다.

&nbsp;

다음 코드를 보자.

&nbsp;

```java
Transaction mostExpensive = transactions.get(0);
if(mostExpensive == null)
  throw new IllegalArgumentException("Empty list of transaction");

for(Transaction t : transactions.subList(1, transactions.size())){
  if(t.getValue() > mostExpensive.getValue()){
    mostExpensive = t;
  }
}
```

&nbsp;

> 위와 같이 '무엇을'에 집중하는 방식을 선언형 프로그래밍이라고 부른다. 문제 자체가 코드로 명확히 드러난다는 점이 선언형 프로그래밍의 강점이다.

&nbsp;

### 함수형 프로그래밍이란 무엇인가?

&nbsp;

> 함수형 프로그래밍에서 함수란 수학적인 함수와 같다. 즉, 함수는 0개 이상의 인수를 가지며, 한 개 이상의 결과를 반환하지만 **부작용이 없어야 한다.**

&nbsp;

### 함수형 자바

&nbsp;

> 자바에서는 순수 함수형이 아니라 함수형 프로그램을 구현할 것이다. 실제 부작용이 있지만 아무도 보지 못하게 함으로써 **함수형**을 달성할 수 있다.

&nbsp;

함수형이란 또한 함수나 메서드가 어떠한 예외도 일으키지 않아야 한다.

&nbsp;

### 참조 투명성

&nbsp;

> '부작용을 감춰야 한다'라는 제약은 **참조 투명성**개념으로 귀결된다.

참조 투명성은 비싸거나 오랜 시간이 걸리는 연산을 기억화 또는 캐싱을 통해 다시 계산하지 않고 저장하는 최적화 기능을 제공한다.

&nbsp;

### 재귀와 반복

&nbsp;

> 순수 함수형 프로그래밍 언어에서는 while, for과 같은 반복문을 포함하지 않는다. 이는 이러한 반복문 때문에 변화가 자연스럽게 코드에 스며들 수 있기 때문이다.

&nbsp;

일반적으로 반복 코드보다 재귀 코드가 더 비싸다는 의견이 있다. 이는 재귀함수를 호출할 때마다 호출 스택에 각 호출시 생성되는 정보를 저장할 새로운 스택 프레임이 만들어지기 때문일 것이다.

```java
// recursive factorial

static long factorialRecursive(long n){
  return n == 1 ? 1 : n * factorialRecursive(n-1);
}
```

하지만 재귀 코드가 안좋기만 할까? 그것은 틀렸다. 함수형 언어에서는 **꼬리 호출 최적화**라는 해결책을 제공하기 때문이다.

&nbsp;

```java
// tail-call optimization version

static long factorialTailRecursive(long n){
  return factorialHelper(1, n);
}

static long factorialHelper(long acc, long n){
  return n == 1 ? acc : factorialHelper(acc * n, n - 1);
}
```

> 위와 같은 최적화 코드는 일반 재귀와 달리 꼬리 재귀에서는 컴파일러가 하나의 스택 프레임을 재활용할 가능성이 생긴다.

&nbsp;

안타깝게도 자바는 최적화를 제공하지 않는다. 그럼에도 여전히 고전적인 재귀보다는 여러 컴파일러 최적화 여지를 남겨둘 수 있는 꼬리 재귀를 적용하는 것이 좋다. 스칼라, 그루비 같은 최신 JVM 언어는 이와 같은 재귀를 반복으로 변환하는 최적화를 제공한다.

&nbsp;

## chapter 19 - 함수형 프로그래밍 기법

&nbsp;

일반값처럼 취급할 수 있는 함수를 **일급 합수**라고 한다.

### 고차원 함수

&nbsp;

> 함수형 프로그래밍 커뮤니티에 따르면 Comparator.comparing처럼 다음 중 하나 이상의 동작을 수행하는 함수를 고차원 함수라 부른다.

- 하나 이상의 함수를 인수로 받음
- 함수를 결과로 반환

&nbsp;

### 커링

&nbsp;

> 커링이란 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다.

&nbsp;

### 영속 자료구조

&nbsp;

> 함수형 프로그램에서는 함수형 자료구조, 불변 자료구조 등의 용어도 사용하지만 보통은 **영속 자료구조**라고 부른다.

&nbsp;

영속 자료구조를 만들기 위해 **'결과 자료구조**를 바꾸지 말라'는 것이 자료구조를 사용하는 모든 사용자에게 요구하는 단 한 가지 조건이다.

&nbsp;

### 스트림과 게으른 평가

&nbsp;

> 이전 장들에서 스트림은 데이터 컬렉션을 처리하는 편리한 도구를 살펴봤다.효율적인 구현 및 여러 이유로 자바 8 설계자들은 스트림을 조금 특별한 방법으로 자바 8에 추가했다.

```java
// 재귀 스트림
public static Stream<Integer> primes(int n){
  return Stream.iterate(2, i -> i + 1)
    .filter(MyMathUtils::isPrime)
    .limit(n);
}

public static boolean isPrime(int candidate){
  int candidateRoot = (int) Math.sqrt((double) candidate);
  return IntStream.rangeClosed(2, candidateRoot)
    .noneMatch(i -> candidate % i == 0);
}
```

위의 코드는 우선 후보 수로 정확히 나누어떨어지는지 매번 모든 수를 반복 확인했으므로 좋은 코드는 아니다.

&nbsp;

```java
// 게으른 자료구조 구현
static Intstream numbers(){
  return IntStream.iterate(2, n -> n + 1);
}

static int head(IntStream numbers){
  return numbers.findFirst().getAsInt();
}

static IntStream tail(IntStream numbers){
  return numbers.skip(1);
}

IntStrean numbers = numbers();
int head = head(numbers);
IntStream filtered = tail(numbers).filter(n -> n % head != 0);

 public MyList<T> filter(Predicate<T> p) {
    return isEmpty() ?
       this : p.test(head()) ?
              new LazyList<>(head(), () -> tail().filter(p)) :
              tail().filter(p);
}
```

### 패턴 매칭

&nbsp;

> 일반적으로 함수형 프로그래밍을 구분하는 또 하나의 중요한 특징으로 (구조적인) 패턴 매칭을 들 수 있다.

&nbsp;

## chapter 20 - OOP와 FP의 조화 : 자바와 스칼라 비교

&nbsp;

> 스칼라는 객체지향과 함수형 프로그래밍을 혼합한 언어다.

&nbsp;

```scala
var authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" -> 53)
```

이와 같이 스칼라에서는 간단하게 컬렉션을 만들 수 있다.

명시적으로 형변환을 지정하지 않아도 스칼라는 자동으로 변수형을 추론하는 기능이 있다. 즉, 모든 변수의 형식은 컴파일을 할 때 결정된다,

&nbsp;

### 옵션

&nbsp;

옵션이라는 자료구조도 있다. 스칼라의 Option은 10장에서 살펴본 자바의 Optional과 같은 기능을 제공한다. 즉, 사용자는 메서드의 시그니처만 보고도 Optional값이 반환될 수 있는지 여부를 알 수 있다. null 대신 Optional을 사용하면 null 포인터 예외를 방지할 수 있다.

&nbsp;

### 함수

&nbsp;

> 스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹이다. 명령어 그룹을 쉽게 추상화할 수 있는 것도 함수 덕분이며 동시에 함수는 함수형 프로그래밍의 중요한 기초석이다.

&nbsp;

### 클래스와 트레이트

> 스칼라의 트레이트는 자바의 인터페이스를 대체한다. 트레이트로 추상 메서드와 기본 구현을 가진 메서드 두 가지를 모두 정의할 수 있다.

&nbsp;

## chapter 21 - 결론 그리고 자바의 미래

&nbsp;

### 구체화된 제네릭

> 자바 5에서 제네릭을 소개했을 때 제네릭이 기존 JVM과 호환성을 유지해야 했다. 결과적으로 ArrayList<Integer>이나 ArrayList<String> 모두 런타임 표현이 같게 되었다.

이를 **제네릭 다형성의 삭제 모델**이라고 한다.

### 값 형식

&nbsp;

> 객체지향 프로그래밍에 객체가 필요한 것처럼 함수형 프로그래밍에 값 형식이 도움을 준다.

&nbsp;

#### 컴파일러가 Integer와 int를 같은 값으로 취급할 수는 없을까?

&nbsp;

> 암묵적 박싱과 언박싱이 자바 1.1 이후로 서서히 퍼져나갔음을 생각해보면 이제 그냥 단순하게 자바에서 객체형값과 기본형값을 같은 값으로 취급할 때가 된 것 아닐까라는 생각을 해볼 수 있다.

이는 괜찮은 아이디어지만 실현이 쉽지 않다.

기본형에서는 비트를 비교해서 같음을 판단하지만 객체에서는 참조로 같음을 판단한다.

&nbsp;

## Appendix A

&nbsp;

### 어노테이션

&nbsp;

- 어노테이션을 반복할 수 있다.
- 모든 형식에 어노테이션을 사용할 수 있다.

> 자바의 어노테이션은 부가 정보를 프로그램에 장식할 수 있는 기능이다. 즉, 어노테이션은 **문법적 메타데이터**다.

&nbsp;

## Appendix B

&nbsp;

### 컬렉션

&nbsp;

- 맵 : 키에 매핑되는 값이 있는지 여부를 확인하는 기존의 get 메서드를 getOrDefault로 대신할 수 있다. getOrDefault는 키에 매핑되는 값이 없으면 기본값을 반환한다.

- 컬렉션 : removeIf 메서드로 프레디케이트와 일치하는 모든 요소를 컬렉션에서 제거할 수 있다.

- 리스트 : replaceAll 메서드는 리스트의 각 요소를 주어진 연산자를 리스트에 적용한 결과로 대체한다.

- Collections 클래스 : Collections는 불변의, 동기화된, 검사된, 빈 NavigableMap과 NavigableSet을 반환할 수 있는 새로운 메서드를 포함한다. 또한 동적 형식 검사에 기반을 둔 Queue 뷰를 반환하는 checkedQueue라는 메서드도 제공한다.

- Comparator : Comparator 인터페이스는 디폴트 메서드와 정적 메서드를 추가로 제공한다.

&nbsp;

### 동시성

&nbsp;

- 아토믹 : 단일 변수에 아토믹 연산을 지원하는 숫자 클래스를 제공한다.

- Adder와 Accumulator : 자바 API는 여러 스레드에서 읽기 동작보다 갱신 동작을 많이 수행하는 상황이라면 Atomic 클래스 대신 Adder, Accumulator를 사용하라고 권고한다.

- ConcurrentHashMap : ConcurrentHashMap은 동시 실행 환경에 친화적인 새로운 해시맵이다. ConcurrentHashMap은 내부 자료구조의 일부만 잠근 상태로 동시 덧셈이나 갱신 작업을 수행할 수 있는 기능을 제공한다.

&nbsp;

### Arrays

&nbsp;

> Arrays 클래스는 배열을 조작하는 데 사용되는 다양한 정적 메서드를 제공한다.

- parallelSort : 해당 메서드는 자연 순서나 객체 배열의 추가 Comparator를 사용해 특정 배열을 병렬로 정렬하는 기능을 수행한다.

- setAll, parallelSetAll 사용하기 : setAll은 지정된 배열의 모든 요소를 순차적으로 설정하고 parallelSetAll은 모든 요소를 병렬로 설정한다. 참고로, parallelSetAll은 병렬로 실행되므로 parallelSetAll에 전달하는 함수는 부작용이 없어야 한다.

&nbsp;

## Appendix C - 스트림에 여러 연산 병렬로 실행하기

> 스트림에서는 한 번만 연산을 수행할 수 있으므로 결과도 한 번만 얻을 수 있다는 것이 자바 8 스트림의 가장 큰 단점이다.

&nbsp;

### 스트림 포킹

&nbsp;

> 스트림에서 여러 연산을 병렬로 실행하려면 먼저 원래 스트림을 감싸면서 다른 동작을 정의할 수 있도록 StreamForker를 만들어야 한다.

```java
public class StreamForker<T> {
private final Stream<T> stream;
private final Map<Object, Function<Stream<T>, ?>> forks = new HashMap<>();
public StreamForker(Stream<T> stream) {
  this.stream = stream;
}
public StreamForker<T> fork(Object key, Function<Stream<T>, ?> f) {
    forks.put(key, f);
    return this;
}

public Results getResults() {
// 구현해야 함
}

```

fork는 두 인수를 받는다.

- 스트림을 특정 연산의 결과 형식으로 변환하는 Function
- 연산의 결과를 제공하는 키. 내부 맵에키/함수 쌍 저장

여기서 사용자는 스트림에서 세 가지 키로 인덱스된 세 개의 동작을 정의한다. StreamForker는 원래 스트림을 탐색하면서 세 개의 스트림으로 포크시킨다. 이제 포크된 스트림에 세 연산을 병렬로 적용할 수 있으며 결과 맵에 각 키의 인덱스를 이용해서 함수 적용 결과를 얻을 수 있다.

&nbsp;

### 성능 문제

&nbsp;

> 한 번만 스트림을 활용하는 것이 성능이 좋을지, 스트림을 여러 번 탐색하는 것이 좋은지 판단하는 가장 좋은 방법은 '직접 측정'해보는 것이다!

&nbsp;

## Appendix D - 람다와 JVM 바이트 코드

&nbsp;

### 익명 클래스

&nbsp;

> 람다 표현식은 함수형 인터페이스의 추상 구현을 제공하므로 자바 컴파일러가 컴파일을 할 때 람다 표현식을 익명 클래스로 변환할 수 있다면 문제가 해결될 것 같지만, 익명 클래스는 애플리케이션 성능에 악영향을 주는 특성을 포함한다.

- 컴파일러는 익명 클래스에 대응하는 새로운 클래스 파일을 생성한다.

&nbsp;

### 바이트 코드 생성

&nbsp;

> 자바 컴파일러는 자바 소스 파일을 자바 바이트코드로 컴파일한다. JVM은 생성된 바이트코드를 실행하면서 애플리케이션을 동작시킨다. 익명 클래스와 람다 표현식은 각기 다른 바이트코드 명령어로 컴파일된다.

&nbsp;

### 구원투수 InvokeDynamic

&nbsp;

> JVM의 동적 형식 언어를 지원할 수 있도록 JDK7에 InvokeDynamic이라는 명령어가 추가되었다. invokeDynamic은 메서드를 호출할 때 더 깊은 수준의 재전송과 동적 언어에 의존하는 로직이 대상 호출을 결정할 수 있는 기능을 제공한다.

이를 사용하면 다음과 같은 장점을 얻을 수 있다.

- 람다 표현식의 바디를 바이트 코드로 변환하는 작업이 독립적으로 유지된다.

- 람다 덕분에 추가적인 필드나 정적 초기자 등의 오버헤드가 사라진다.

- 상태 없는 람다에서 람다 객체 인스턴스를 만들고, 캐시하고, 같은 결과를 반환할 수 있다.

- 람다를 처음 실행할 때만 변환과 결과 연결 작업이 실행되므로 추가적인 성능 비용이 들지 않는다.

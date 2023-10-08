# Chapter 18 : 함수형 관점으로 생각하기
## 18.1 : 시스템 구현과 유지보수
- 구조 평가 방법
	- 결합성 : 상호 읜존성
	- 응집성 : 서로 어떤 관계를 갖는지
- 함수형 프로그래밍의 특징
	- 부작용 없음 : 입력을 받고 결과를 리턴하는 것 외의 다른 작용을 하지 않는다.
	- 불변성 : 같은 입력값에 대해 항상 같은 결과를 반환한다.

### 공유된 가변 데이터
공유되는 가변 데이터가 있다면 유지보수가 어렵다.

##### 순수 메서드 (부작용 없는 메서드)
- 자신을 포함하는 클래스의 상태 를 바꾸지 않음
- 다른 객체의 상태를 바꾸지 않음
- return문을 통해서만 자신의 결과를 반환


### 선언형 프로그래밍
- 스트림의 내부 반복과도 같이 **'무엇을'에 집중하는 방식**

### 함수형 프로그래밍
- 선언형 프로그래밍을 따르는 대표적인 방법

## 18.2 : 함수형 프로그래밍이란 무엇인가?
함수형 프로그래밍에서의 '함수'란 수학적인 함수와 같다.
- 0개 이상의 인수를 가진다
- 한 개 이상의 결과를 반환한다.
- 부작용이 없어야 한다.
	- 부작용 : 다른 객체의 필드를 고치는 등의 작업

### 함수형 자바
자바에서는 순수 함수형이 아니라 함수형 프로그램을 구현할 것
> 실제 부작용이 있지만 이를 아무도 보지 못하게 함으로써 **함수형** 을 달성할 수 있다.
> - 진입할 때 어떤 필드의 값을 증가시켰다가 빠져 나올 때 필드의 값을 돌려놓는 등
> - 하지만 다른 스레드가 해당 필드의 값을 확인 혹은 메서드를 호출 한다면 함수형이 아니게 된다.
> 	- 메서드의 바디를 잠글 수 있지만, 병렬성을 잃게 된다.

**수학형 함수**는 주어진 인수값에 대응하는 하나의 결과를 반환.

### 참조 투명성
- 함수는 어떤 입력이 주어졌을 때 언제, 어디서 호출하든 같은 결과를 생성해야 한다.
	- 예를 들면...
	- "raoul".replace('r', 'R')
	- 두 개의 final int 변수를 더하는 연산
- 장점 
	1. 프로그램 이해에 큰 도움을 준다.
	2. 비싸거나 오랜 시간이 걸리는 연산을 **기억화** 또는 **캐싱**을 통해 최적화 가능
- 문제
	- 같은 요소를 포함하는 다른 두 개의 리스트 객체를 반환한다면 **투명한 메서드가 아니다**
	- 결과 리스트를 (불변의) 순수값으로 사용할 것이라면 참조적으로 투명하다고 여길 수 있다.

### 함수형 프로그래밍
```java
static List<List<Integer>> concat(List<List<Integer>> a,
								  List<List<Integer>> b) {
	a.addAll(b);
	return a;
}
```
- 위와 같이 함수를 구현하면 a의 객체를 바꾸게 되므로
```java
static List<List<Integer>> concat(List<List<Integer>> a,
								  List<List<Integer>> b) {
	List<List<Integer>> r = new ArrayList<>(a); 
	r.addAll(b);
	return r;
}
```
- 위와 같이 함수를 구현하는 것이 더 좋은 코드이다.

## 18.3 재귀와 반복
> 순수 함수형 프로그래밍 언어에서는 while, for 같은 반복문을 포함하지 않는다.
> 원하지 않는 변화가 자연스럽게 코드에 스며들 수 있기 때문이다.
> 이론적으로 반복을 이용하는 모든 프로그램은 재귀로도 구현할 수 있는데, 제귀를 이용하면 변화가 일어나지 않는다.

- 팩토리얼을 재귀로 구현한다면 아래와 같이 구현할 수 있다.
```java
static long factorial(long n) {
	return n == 1 ? 1 : n * factorial(n-1);
}
```
- 스트림으로 좀 더 단순하게 팩토리얼을 정의할 수도 있다.
```java
static long factorial(long n) {
	return LongStream.rangeClosed(1, n)
					 .reduce(1, (long a, long b) -> a * b);
}
```

재귀의 단점
1. 함수를 호출할 때마다 호출 스택에 각 호출시 생성되는 정보를 저장할 새로운 스택 프레임이 만들어진다.
	- 입력값에 비례해서 메모리 사용량이 증가한다.
2. 따라서, 큰 입력값을 사용하면 StackOverflowError가 발생한다.

함수형 언어에서는 해결책으로 **꼬리 호출 최적화** 제공
```java
static long factorialTailRecursive(long n) {
	return factorialHelper(1, n);
}
static long factorialHelper(long acc, long n) {
	return n == 1 ? acc : factorialHelper(acc * n, n-1);
}
```
- 재귀 호출이 가장 마지막에 이루어진다.
- 재귀 스택을 계속 쌓는 것이 아니라, **단일 스택 프레임을 재사용한다.**
<br>

- ***자바는 이와 같은 최적화를 제공하지 않는다.***
- 최신 jvm 언어는 꼬리재귀를 반복으로 변환하는 최적화를 제공한다.
---

# Chapter 19 : 함수형 프로그래밍 기법
## 19.1 : 함수는 모든 곳에 존재한다.
### 고차원 함수

조건
1. 하나 이상의 함수를 인수로 받음
2. 함수를 결과로 반환

고차원 함수란  ***위의 조건 중 하나 이상을 동작을 수행하는 함수***

예시) 연산 파이프라인
```java
Function<String, String> transformationPipeline =
	addHeader.andThen(Letter::checkSpelling)
			 .andThen(Letter::addFooter);
```

> 고차원 함수를 적용할 때도, 부작용을 포함하는 함수를 사용하면 문제가 발생한다.
> 어떤 인수가 전달 될 지 알 수 없으므로 인수가 부작용을 포함할 가능성을 염두해 두어야 한다.
> 부작용을 포함하지 않는다면 가장 좋지만 포함한다면 정확하게 문서화하는 것이 좋다.

### 커링
- 함수를 모듈화하고 코드를 재사용하는 데 도움을 준다.
> `x * f + b`라는 **변환**을 수행하는 하나의 함수를 `fun1(x, f, b)`라고 한다면
> `(double x) -> x * f + b`를 반환하는 함수 `fun2(f, b)`로 세분화하여
> 다른 비슷한 **변환**에 재사용 할 수 있게 한다.
> 이후 `(fun2(f, b))(x)` 와 같은 방식으로 사용할 수 있다.
> - 코드의 모양이 같다는 것이 아니다.
> - `fun2`를 통해 반환받은 함수에 `x`를 인자로 주고 실행시킨다는 뜻


## 19.2 : 영속 자료구조
함수형 자료구조, 불변 자료구조를 보통 **영속 자료구조** 라고 부른다.

### 파괴적인 갱신과 함수형
> 여행지를 연결하기 위한 class를 구현한다고 가정하자.
> a 라는 인스턴스가 `서울 -> 구미` 를 연결하고
> b 라는 인스턴스가 `구미 -> 부산` 을 연결한다면
> 
> append라는 함수를 만들어 이 두 여정을 연결할 때
> a 인스턴스에 직접적으로 `부산`을 연결한다면, a 인스턴스를 사용하던 클라이언트들은 문제에 빠지게 된다.
> -> 이를 ***파괴적인 갱신*** 이라고 한다.
> 
> 함수형으로 append를 구현한다면
> a 인스턴스를 복사하여 b를 연결하여 반환하게 할 수 있다.
> **함수형은 자료구조를 갱신하지 않는다!**

- 비슷한 다른 자료구조형 예제 : Tree

> 값을 업데이트 할 때, 직접적으로 업데이트 하지 않고,
> new Tree를 만들어 새로운 트리를 반환하게 한다.

<br>

***파괴적인 갱신 대신 함수형 메서드를 만든다면 ***
- 새로운 자료구조를 만듦
- **인수를 이용해서 가능한 많은 정보를 공유**
- 이와 같은 함수형 자료구조를 ***영속*** 이라고 한다.

## 19.3 스트림과 게으른 평가
### 자기 정의 스트림
##### 스트림의 제약사항
- 스트림은 한 번만 소비할 수 있다.
- 따라서 재귀적으로 정의할 수 없다.
##### 스트림의 문제점 (코드)
```java
// 1.
static IntStream numbers() {
	return IntStream.iterate(2, n -> n + 1);
}
// 2.
static int head(IntStream numbers) {
	return numbers.findFirst().getAsInt();
}
// 3.
static IntStream tail(IntStream numbers) {
	retufn numbers.skip(1);
}
// 4.
static IntStream primes(IntStream numbers) {
	int head = head(numbers);
	return IntStream.concat(
		IntStream.of(head),
		primes(tail(numbers).filter(n -> n % head != 0))
	);
}
```
아리스토테네스의 체 알고리즘을 스트림으로 만들기
1. 소수를 선택할 숫자 스트림이 필요하다.
2. 스트림의 머리를 가져온다.
3. 스트림의 꼬리에서 가져온 수로 나누어떨어지는 모든 수를 걸러 제외시킨다.
4. 남은 숫자만 포함하는 새로운 스트림에서 소수 찾기. (1번 과정부터 다시 반복하므로 재귀)

**위의 4번째 코드의 문제점**
- 에러 : 최종연산(findFirst, skip등)을 호출하면 스트림이 완전 소비된다.
- 무한 재귀 : primes를 직접 재귀적으로 호출

##### 게으른 평가
위의 primes를 무한 재귀 호출하는 문제를 해결하는 방법으로 **게으른 평가( 비엄격한 평가, 이름에 의한 호출)**이 있다.
- concat의 두 번째 인수에서 primes를 게으르게 평가하는 방식

### 게으른 리스트 만들기
**`Supplier<T>`**를 이용하면 꼬리가 모두 메모리에 존재하지 않게 할 수 있다.
```java
class LazyList<T> implements MyList<T> {
    final T head;
    final Supplier<MyList<T>> tail;
    public LazyList(T head, Supplier<MyList<T>> tail) {
	this.head = head;
	this.tail = tail; }
	public T head() {
	    return head;
	}
	public MyList<T> tail() {
	    return tail.get();
	}
	public boolean isEmpty() {
		return false;
	}
}
```
- Supplier의 get 메서드를 호출하면 LazyList의 노드가 만들어진다.

##### 게으른 리스트 : 소수 생성
```java
public static MyList<Integer> primes(MyList<Integer> numbers) {
    return new LazyList<>(
	    numbers.head(),
        () -> primes(
			numbers.tail()
				   .filter(n -> n % numbers.head() != 0)
		)
	);
}
// 게으른 필터 구현
public MyList<T> filter(Predicate<T> p) {
	return isEmpty() ?
	    this : // 새로운 Empty<>()를 반환할 수도 있지만 여기서는 this로 대신할 수 있다.
        p.test(head()) ?
			new LazyList<>(head(), () -> tail().filter(p)) :
			tail().filter(p);
}
```

##### 게으른 리스트 : 모든 소수 출력
```java
// 함수형이므로 재귀적으로 구현
static <T> void printAll(MyList<T> list) {
	if (list.isEmpty())
	   return;
	System.out.println(list.head());
	printAll(list.tail());
}
```
- 자바는 꼬리 호출을 지원하지 않으므로 무한히 출력되기 전에 스택 오버플로우가 발생한다.

## 19.4 패턴 매칭
- 함수형 프로그래밍을 구분하는 또 하나의 중요한 특징

##### 방문자 디자인 패턴
- 지정된 데이터 형식의 인스턴스를 입력으로 받는다.
```java
class BinOp extends Expr {
       ...
       public Expr accept(SimplifyExprVisitor v) {
           return v.visit(this);
	}
}

public class SimplifyExprVisitor {
       ...
    public Expr visit(BinOp e) {
        if("+".equals(e.opname) && e.right instanceof Number && ...) {
            return e.left;
        }
		return e;
	}
}
```

### 자바로 패턴 매칭 흉내 내기
```java
interface TriFunction<S, T, U, R> {
       R apply(S s, T t, U u);
}
static <T> T patternMatchExpr(
					   Expr e,
					   TriFunction<String, Expr, Expr, T> binopcase,
					   Function<Integer, T> numcase,
					   Supplier<T> defaultcase) {
	return
		(e instanceof BinOp) ?
			binopcase.apply(((BinOp)e).opname, ((BinOp)e).left,
											  ((BinOp)e).right) :
		(e instanceof Number) ?
			numcase.apply(((Number)e).val) :
			defaultcase.get();
}
// ====================다음과 같이 호출 가능========================================
patternMatchExpr(e, (op, l, r) -> {return binopcode;},
                    (n) -> {return numcode;},
					() -> {return defaultcode;})
// ============================================================================
public static Expr simplify(Expr e) {
	TriFunction<String, Expr, Expr, Expr> binopcase = // BinOp표현식처리
		(opname, left, right) -> {
		if ("+".equals(opname)) { // 더하기처리
			if (left instanceof Number && ((Number) left).val == 0) { 
				return right;
			}
			if (right instanceof Number && ((Number) right).val == 0) {
				return left;
			}
		}
		if ("*".equals(opname)) { // 곱셈처리
			if (left instanceof Number && ((Number) left).val == 1) { 
				return right;
			}
			if (right instanceof Number && ((Number) right).val == 1) {
				return left;
		    }
		}
	    return new BinOp(opname, left, right);
	};
	Function<Integer, Expr> numcase = val -> new Number(val); // 숫자처리
	Supplier<Expr> defaultcase = () -> new Number(0);
	// 수식을 인식할 수 없을 때 기본 처리
	return patternMatchExpr(e, binopcase, numcase, defaultcase);
	// 패턴 매칭 적용
}
// ====================다음과 같이 호출 가능========================================
Expr e = new BinOp("+", new Number(5), new Number(0));
Expr match = simplify(e);
System.out.println(match);
// ============================================================================
```

### 19.5 기타 정보
##### 캐싱 또는 기억화
> 예를 들어,
> 트리 형식의 토폴로지를 갖는 네트워크 범위 내에 존재하는 노드의 수를 계산하는 메서드가 있다고 가정
> 네트워크는 다행히 불변이지만, 구조체를 재귀적으로 탐색해야 하므로 노드 계산 비용이 비싸다.
> 게다가 이와 같은 계산을 반복해서 수행해야 할 것 같다.
> 참조 투명성이 유지된다면 (같은입력->같은출력) **기억화(memorization)** 가능

---
# Chapter 20 : OOP와 FP의 조화 : 자바와 스칼라 비교
> 스칼라는 객체지향과 함수형 프로그래밍을 혼합한 언어.
> JVM에서 수행된다.
> 자바에 비교하면 스칼라는
> - 복잡한 형식 시스템
> - 형식추론
> - 패턴 매칭
> - 도메인 전용 언어를 단순하게 정의할 수 있는 구조
> 
> 들을 제공한다.

## 20.1 스칼라 소개
### Hello beer
0. 원하는 출력 내용
```
Hello 2 bottles of beer
Hello 3 bottles of beer
Hello 4 bottles of beer
Hello 5 bottles of beer
Hello 6 bottles of beer
```
1. 함수형 자바
```java
public class Foo {
	public static void main(String[] args) {
		IntStream.rangeClosed(2, 6)
				 .forEach(n -> System.out.println("Hello " + n + 
												  "bottles of beer"));
	}
}
```
2. (함수형) 스칼라
```scala
object Beer {
	def main(args: Array[String]) {
		2 to 6 foreach { n => println(s"Hello ${n} bottles of beer") }
	}
}
```
- 스칼라에서는 모든 것이 객체이다. **(기본형이 없다)**
- object로 직접 **싱글턴 객체**를 만들 수 있다.
	- main 메서드에 static이 없는 것도 이 때문이다.
- ;은 선택사항이다.
- 문자열 보간법 (문자열 자체에 변수와 표현식을 바로 삽입하는 기능)을 사용
- 인픽스 형식으로 한줄로 `2 to 6`으로 범위를 반환하는 함수를 사용했다.
	- 원형은 2.to(6)
- `->` 대신 `=>`를 사용했다.

### 기본 자료구조 : 리스트, 집합, 맵, 튜플, 스트림, 옵션
```scala
// 맵
val authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" -> 53)
// 단방향 연결 리스트
val autors = List("Raoul", "Mario", "Alan")
// 셋
val numbers = Set(1, 1, 2, 3, 5, 8)
```
- 지금까지 만든 컬렉션들은 기본적으로 ***불변***이다
- 갱신은 이전에 배운 **영속**이라는 용어를 떠올리자
	- 새로운 객체를 만드는 방법으로 자료구조를 갱신한다.
 
```scala
val numbers = Set(2, 5, 3);
val newNumbers = numbers + 8
// + 8 연산이 기존의 numbers 셋에 영향을 주지 않는다.
```

##### 컬렉션 사용하기
- 스칼라의 컬렉션 동작은 스트림 API와 비슷하다. 예를 들면 map과 filter를 사용할 수 있다.
```scala
val fileLines = Source.fromFile("data.txt").getLines.toList()
val linesLongUpper =
	fileLines.filter(l => l.length > 10)
			 .map(l => l.toUpperCase())
```
- 첫 번째 행은 파일의 모든 행을 문자열 행으로 변환한다.
- 두 번째 행은 연산의 파이프라인을 생성한다.

```scala
val linesLongUpper = 
	fileLines filter (_.length() > 10) map(_.toUpperCase())
```
- 위와 같이 인픽스를 이용해 더 간소화 할 수도 있다.

```scala
val linesLongUpper =
	fileLines.par filter (_.length() > 10) map(_.toUpperCase())
```
- 자바의 parallel과 같이 스칼라도 pal을 이용해 병렬 스트림을 만들 수 있다.

###### 튜플
```scala
val raoul = ("Raoul", "+ 44 887007007")
val alan = ("Alan", "+44 883133700")
println(raoul._1)
```
- 스칼라는 임의 크기의 튜플을 제공한다.
- 여러 종류의 객체를 넣을 수도 있다.
- 접근자는 1부터 시작한다.

###### 스트림
- 게으르게 평가되는 자료구조
- 자바보다는 느리다. '캐시'되어야 하기 때문

###### 옵션
- 자바의 Optional
- 자바와의 호환성 때문에 scala에도 null이 존재하지만 사용하지 않기를 권장한다.

## 20.2 함수
- 함수 형식 : 자바의 디스크럽터
- 익명 함수 : 자바의 람다와 달리 비지역 변수 기록에 제한을 받지 않는다.
- 커링 지원 : `f(a, b, c)` -> `g(b, c)(a)`

### 스칼라의 일급 함수
- 스칼라에서 함수는 일급 객체이다
- 필터 함수의 시그니처
	- 파라미터로 함수를 받는 예시
```
def filter[T](p: (T) => Boolean): List[T]
```

### 익명 함수와 클로저
- 람다 표현식과 비슷한 문법 제공
- 스칼라의 익명 함수는 클로저에 해당한다.
	- 클로저 : 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스
	- 스칼라의 익명 함수는 값이 아니라 변수를 캡처할 수 있다.
		- `var count = 0` 이후에 `count+=1` 가능
  
### 커링
- 커링을 하기 위한 특수문법 제공
```scala
def multiplyCurry(x :Int)(y :Int) = x * y
var r1 = multiplyCurry(2)(10)
var multiplyByTwo : Int => Int = multiplyCurry(2)
var r2 = multiplyByTwo(10)
```

## 20.3 클래스와 트레이트

### 간결성을 제공하는 스칼라의 클래스
- 게터와 세터 등이 암시적으로 생성된다.

### 스칼라 트레이트와 자바 인터페이스
- 트레이트 : 자바의 인터페이스
	- 추상 메서드와 기본 구현을 가진 메서드 두 가지 모두 정의 가능
	- 다중 상속 지원
		- 상태 또한 다중 상속할 수 있다
	- 인스턴스화 과정에서도 상속이 가능하다
		- 다만 컴파일 할 때 조합 결과가 결정된다.
```scala
trait Sized {
	var size : Int = 0
	def isEmpty() = size == 0
}
class Empty extends Sized
println(new Empty().isEmpty()) // true를 출력한다.

// 인스턴스화 과정에서도 조합이 가능하다!!!
class Box
val b1 = new Box() with Sized
println(b1.isEmpty())
```


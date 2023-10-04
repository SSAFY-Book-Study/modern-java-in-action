# part6. 함수형 프로그래밍과 자바 진화의 미래

### 1. 함수형 관점으로 생각하기

### 1.1 시스템 구현과 유지보수

1. **공유된 가변 데이터**

- 문제 : 실질적으로 많은 프로그래머가 유지보수 중 코드 크래시 디버깅 문제를 많이 겪는다.
- 원인 : 예상하지 못한 변숫값.
- 예상하지 못한 변숫값의 원인 :
    - 프로그램이 공유 가변 데이터 구조를 사용 ⇒ 데이터 갱신 사실을 추적하기 어려워짐.
- 해결 :
    - 순수 메서드 or 부작용 없는 메서드 사용.
        - 자신을 포함하는 클래스의 상태 & 다른 객체의 상태 변경 X.
        - return 문을 통해서만 자신의 결과 반환.
    - 불변 객체 사용.
        - 인스턴스화한 다음에는 객체의 상태 변경 X.
- 결과 : 메서드가 서로 간섭하지 X ⇒ 잠금을 사용하지 않고도 멀티코어 병렬성 사용 가능(스레드 안전성↑).

---

1. **선언형 프로그래밍**

- 프로그램 구현 방식 구분 :


    1) 명령형 프로그래밍
    
    - “어떻게” 에 집중
    - 고전의 객체지향 프로그래밍 코드 방식
    - 할당, 조건문, 분기문, 루프 등
    
    2) 선언형 프로그래밍
    
    - “무엇을” 에 집중.
    - 구현 방법을 라이브러리가 결정(내부 반복)
    - 스트림 API, 람다
- 선언형 프로그래밍의 장점 : 문제 자체가 코드로 명확하게 드러난다.

---

1. **왜 함수형 프로그래밍인가?**

- 함수형 프로그래밍 : 선언형 프로그래밍 + 부작용을 멀리한다.
- 장점 :
    - 쉽게 시스템을 구현하고 유지보수한다, 부작용이 없다.
    - 람다 : 작업을 조합하거나, 동작을 전달하는 등의 기능.
    - 스트림 : 여러 연산을 연결해서 복잡한 질의 표현.

---

### 1.2 함수형 프로그래밍이란 무엇인가?

1. **함수형 자바**

- 순수 함수형 프로그래밍 : 함수 & if-then-else 등의 수학적 표현만 사용하는 방식.
- 함수형 프로그래밍 : 시스템의 다른 부분에 영향 X, 내부적으로는 함수형이 아닌 기능도 사용.
- 자바의 함수형 프로그래밍 :
    - 자바로는 완벽한 순수 함수형 프로그래밍 구현 X ⇒ 시스템의 컴포넌트가 순수한 함수형인 것처럼 동작하도록 구현가능(함수형 프로그래밍).
    - Ex. 진입할 때 어떤 필드의 값을 증가시켰다가 나올때 돌려놓는 부작용 없는 함수 or 메서드.
        - 단일 스레드 : 부작용 없으므로 함수형 프로그래밍 O.
        - 다중 스레드 : 동작 도중에는 필드값이 변경되기 때문에 함수형 프로그래밍 X.
        - 다중 스레드 + Lock : 함수형이지만, 실행속도가 느려짐.
- 함수형 프로그래밍 규칙 :
    - 객체의 모든 필드가 final.
    - 모든 참조 필드는 불변 객체를 직접 참조.
    - 예외적으로 메서드 내에서 생성한 객체의 필드는 갱신 가능.
        - 단, 새로 생성한 객체의 필드 갱신은 외부에 숨겨야 하고, 몇번을 호출해도 결과는 동일해야 함.
    - 어떤 예외도 일으키지 않아야 함.
        - 예외 처리 대신 Optional<T> 활용.
    - 자료구조의 변경을 호출자가 알 수 없도록 감춰야 함.

---

1. **참조 투명성**

- 참조투명성 : 같은 인수로 함수를 호출하면 항상 같은 결과를 반환해야 함.
- 참조투명성 & 최적화 : 비싸거나 오랜시간이 걸리는 연산을 memoization / caching 하는 최적화 기능 제공.
- 자바의 참조 투명성 :
    - Ex. 내부 필드가 참조하는 리스트가 2개일때.
        - 해당 리스트가 가변객체라면, 참조 투명성 X.
        - 해당 리스트가 불변객체라면, 참조 투명성 O.

---

### 1.3 재귀와 반복

- 순수 함수형 프로그래밍 & 반복문 : 반복문 때문에 변화 발생 가능 ⇒ 사용 X.
- 반복문을 대체하는 방법 :
    - 재귀 : 메모리 사용량 증가라는 문제 발생 가능.
    - 꼬리 호출 최적화 : 컴파일러가 하나의 스택 프레임 재활용(최적화).
        - 자바 지원 X.
    - 스트림 : 반복을 스트림으로 대체(코드 단순화).
        - 자바 지원 O.

```java
// 반복 방식의 팩토리얼
int factorialIterative(int n) {
	int r = 1;
	for(int i=1; i<=n; i++) {
		r *= i;
	}
	return r;
}

// 재귀 방식의 팩토리얼
int factorialRecursive(int n) {
	return n == 1 ? 1 : n * factorialRecursive(n-1);
}

// 스트림 방식의 팩토리얼(자바8)
int factorialStream(int n) {
	return IntStream().rangeClosed(1, n)
										.reduce(1, (int a, int b) -> a * b);
}

// 꼬리 호출 최적화(최신 함수형 언어)
int factorialTailRecursive(int n) {
	return factorialHelper(1, n);
}
int factorialHelper(int acc, int n) {
	return n == 1 ? acc : factorialHelper(acc * n, n-1);
}
```

### 2. 함수형 프로그래밍 기법

### 2.1 함수는 모든 곳에 존재한다

1. **고차원 함수**

- 일급 함수 : 일반값처럼 취급할 수 있는 함수.
    - 자바 8부터 일급함수를 지원한다.
    - Ex. 메서드 참조(`::`), 람다 표현식(`()→{}`).
- 고차원 함수 :
    - 하나 이상의 함수를 인수로 받음.
    - 함수를 결과로 반환, 지역변수로 할당, 구조체로 삽입 가능함.
    - 자바 8의 일급 함수도 고차원 함수로 취급 가능.

---

1. **커링**

- 커링 :
    - x, y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g 함수로 대체.
    - g 함수 역시 하나의 인수를 받는 함수 반환.
    - 이때, f(x, y) = (g(x))(y) 성립.
- 예제 :
    - 대부분의 애플리케이션은 국제화 지원, 여기서 단위변환문제 발생 가능.
    - 해당 문제해결을 위해 변환요소(f)와 기준치 측정요소(b) 적용 함수 생성.

```java
// 메서드를 통한 변환 패턴 표현
static double converter(double x, double f, double b) {
	return x * f + b;
}

// 커링을 통한 팩토리 메서드로 변환 패턴 표현 => 인자 개수 줄이고, 재사용 가능한 코드 생성
static DoubleUnaryOperator curriedConverter(double f, double b) {
	return (double x) -> x * f + b;
}

// (1) 변환요소, 기준치 측정요소를 넣어서 변환기 생성
DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
DoubleUnaryOperator convertKmtoMi = curriedConverter(0.6214, 0);

// (2) DoubleUnaryOperator : applyAsDouble 반환, 변환기를 통해 단위 변환
double mi = convertKmtoMi.applyAsDouble(1000);
```

---

### 2.2 영속 자료구조

1. **파괴적인 갱신과 함수형 :**
- 함수형 메서드에서는 전역 자료구조 / 인수로 전달된 자료구조를 갱신할 수 없음.
    - 참조 투명성에 위배, 인수를 결과로 단순하게 매핑할 수 있는 능력이 상실되기 때문.
- 예제 :
    - 여행 구간의 가격 등 상세 정보를 포함하는 TrainJourney 클래스의 단방향 연결 리스트.
    - (1)의 문제점 : 파괴적인 갱신(firstJourney) 발생 ⇒ 의존하는 코드 동작 X.
    - (2)의 해결책 : 새로운 자료구조 생성.

```java
// 기차여행을 의미하는 TrainJourney 클래스(단방향 연결 리스트 : 출발지 -> 목적지)
class TrainJourney {
	public int price;
	public TrainJourney onward;
	public TrainJourney(int price, TrainJourney onward) {
		this.price = price;
		this.onward = onward;
	}
}

// (1) X -> Y | Y -> Z
static TrainJourney link(TrainJourney from, TrainJourney to) {
	if(from == null) return to;
	TrainJourney trainJourney = from;
	while(trainJourney.onward != null) trainJourney = trainJourney.onward;
	trainJourney.onward = to;
	return from;
}

// (2) X -> Y | Y -> Z
static TrainJourney append(TrainJourney from, TrainJourney to) {
	return from == null ? to : new TrainJourney(from.price, append(from.onward, to));
}

// 클라이언트 코드
link(firstJourney, secondJourney); // firstJourney(X->Y) | secondJourney(Y->Z)
```

---

1. **트리를 사용한 다른 예제와 함수형 :**
- 예제
    - Tree 자료구조의 노드 데이터 update.
    - (1) update() : 모든 사용자가 같은 자료구조를 공유 ⇒ 누군가 자료구조를 갱신하면 영향 O.
    - (2) fupdate() : 순수한 함수형, 새로운 트리를 만들지만 인수를 이용해서 가능한 많은 정보 공유.

```java
class Tree {
	private String key;
	private int val;
	private Tree left, right;
	public Tree(String key, int val, Tree left, Tree right) {
		this.key = key;
		this.val = val;
		this.left = left;
		this.right = right;
	}
}

class TreeProcessor {
	public static int lookup(String key, int default val, Tree tree) {
		if(tree == null) return defaultval;
		if(key.equals(tree.key)) return tree.val;
		return lookup(key, defaultval, key.compareTo(tree.key) < 0 ? tree.left : tree.right);
	}
}

// (1) 하나의 자료구조로, 메서드가 탐색한 트리를 그대로 반환
public static void update(String key, int newval, Tree tree) {
	if(tree == null) //새로운 노드 추가
	else if(key.equals(tree.key)) tree.val = newval;
	else update(key, newval, key.compareTo(tree.key) < 0 ? tree.left : tree.right);
}

// (2) 함수형 접근법
public static Tree fupdate(String key, int newval, Tree tree) {
	return (tree == null) ? new Tree(key, newval, null, null) :
					key.equals(tree.key) ? new Tree(key, newval, tree.left, tree.right) :
					key.compareTo(tree.key) < 0 ? new Tree(tree.key, tree.val, fupdate(key, newval, tree.left), tree.right) :
					new Tree(tree.key, tree.val, tree.left, fupdate(key, newval, tree.right);
}
```

---

### 2.3 스트림과 게으른 평가

1. 자기 정의 스트림
- 스트림 제약사항 : 재귀적으로 정의 불가.
- 예제 : 스트림 API를 통한 소수구하기 알고리즘 구현 :
    - 재귀적으로 스트림 생성 안되는 이유 : 최종 연산을 호출하면 스트림이 완전 소비됨.

```java
// (1) 스트림 숫자 얻기
public Intstream numbers() {
	return IntStream.iterate(2, n -> n+1);
}

// (2) 1st 요소 얻기
static int head(IntStream numbers) {
	return numbers.findFirst().getAsInt();
}

// (3) 마지막 요소를 통한 필터링
static IntStream tail(IntStrean numbers) {
	return numbers.skip(1);
}
IntStream = numbers = numbers();
int head = head(numbers);
IntStream filtered = tail(numbers).filter(n -> n % head != 0);

// (4) 재귀적으로 소수 스트림 생성
static IntStream primes(IntStream numbers) {
	int head = head(numbers);
	return IntStream,concat(
		IntStream.of(head);
		
	);
}
```

---

1. 게으른 리스트 만들기
- 게으른 평가 :
    - 소수를 처리할 필요가 있을 때만 스트림을 실제로 평가.
    - 일련의 연산을 적용하면 연산히 수행되지 않고, 일단 저장.
    - 최종 연산을 적용해서 실제 계산을 해야하는 상황에서만 실제 연산이 이루어짐.
    - 한번에 여러 연산 처리 가능.

---

### 2.4 패턴 매칭

1. **방문자 디자인 패턴**

- 자바에서는 방문자 디자인 패턴으로 자료형을 언랩할 수 있다.
- 방문자 디자인 패턴 동작 :
    - 방문자 클래스는 지정된 데이터 형식의 인스턴스를 입력받고, 인스턴스의 모든 멤버에 접근한다.

```java
// SimplifyExprVisitor을 인수로 받는 accept를 BinOp에 추가
// BinOp 자신을 SimplifyExprVisitor로 전달

class BinOp extends Expr {
	...
	public Expr accept(SimplifyExprVisitor v) {
		return v.visit(this);
	}
}

// 이제 SimplifyExprVisitor는 BinOp 객체를 언랩할 수 있다

public class SimplifyExprVisitor {
	...
	public Expr visit(BinOp e) {
		if("+".equals(e.opname) && e.right instanceof Number && ... ) {
			return e.left;
		}
		return e;
	}
}
```

---

1. **패턴 매칭의 힘**

- 위와 같은 자바의 방문자 패턴은 패턴 매칭으로 더 단순화할 수 있다.
    - 자바에서는 패턴 매칭을 지원하지 않는다.
    - 람다를 통해 패턴 매칭과 유사한 형태를 만들어볼 수 있다.

```java
// 스칼라의 패턴 매칭
def SimplifyExpression(expr: Expr): Expr = expr match {
	case BinOp("+", e, Number(0)) => e //0더하기
	case BinOp("*", e, Number(1)) => e //1곱하기
	case BinOp("/", e, NUmber(1)) => e //1로나누기
	case _ => expr //expr을 단순화할 수 없다
}
```

### 3. OOP와 FP의 조화 : 자바와 스칼라 비교

### 3.1 스칼라 소개

1. **Hello beer**

- 스칼라 : 객체지향 + 함수형 프로그래밍 언어.
    - 스칼라는 자바에 비해 더 다양하고 심화된 함수형 기능을 제공한다.
- 스칼라 vs 자바 :
    - 스칼라는 기본형이 없다 → Ex. Int 자료형도 객체.
    - 스칼라는 문자열 보간법 사용 가능 → + 연산을 통해 불편하게 코딩 안해도 됨.

```java
// 함수형 자바
public class Foo {
	public static void main(String[] args) {
		IntStream.rangeClosed(2, 6)
			.forEach(n -> System.out.println("Hello "+n+" bottles of beer"));
	}
}

// 함수형 스칼라
object Beer {
	def main(args: Array[String]) {
		2 to 6 foreach {n => println(s"Hello ${n} bottles of beer")}
	}
}
```

---

1. **기본 자료구조 : 리스트, 집합, 맵, 튜플, 스트림, 옵션**

- 컬렉션 만들기 : 스칼라는 간결하게 컬렉션 생성 가능.
- 스칼라 vs 자바(컬렉션 생성) :
    - 자동으로 변수형 추론.
    - 생성되는 컬렉션은 모두 불변.
    - 갱신해야 하는 경우 새로운 컬렉션 생성 ⇒ 언제 어디서 컬렉션을 갱신했는지 신경쓰지 않아도 됨.

    ```java
    // 자바의 컬렉션 생성
    Map<String, Integer> authorsToAge = new HashMap<>();
    authorsToAge.put("Raoul", 23);
    authorsToAge.put("Mario", 40);
    authorsToAge.put("Alan", 53);
    
    // 스칼라의 컬렉션 생성
    val authorsToAge = Map("Raoul" -> 23, "Mario" -> 40, "Alan" -> 53);
    ```


- 컬렉션 사용하기 : 자바의 스트림 API와 비슷하게 동작.
- 스칼라 vs 자바(컬렉션 사용) :
    - 모든 행 → 문자열 행.
    - 연산의 파이프라인 생성.
    - 병렬실행 가능.

- 기타 :
    - 튜플 : 스칼라는 튜플 축약어를 통해 간단하게 튜플 자료형 생성 가능.
    - 스트림 :
        - 기존 계산값 캐싱.
        - 인덱스로 스트림 요소에 접근 가능.
    - 옵션 : 메서드의 시그니처만 보고도 Optional값이 반환될 수 있는지 여부 파악 가능.

### 3.2 함수

1. **스칼라의 일급 함수**

- 스칼라의 함수는 일급값이다.

---

1. **익명 함수와 클로저**

- 스칼라는 익명 함수의 개념을 지원한다.
- 스칼라의 익명 함수(람다표현식과 유사)는 클로저에 해당한다.
    - 클로저 : 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스.
    - 자바의 익명함수 : 비지역 변수의 값을 캡쳐한다 ⇒ final 처럼 만들어버림.
    - 스칼라의 익명함수 : 비지역 변수를 캡쳐한다 ⇒ 비지역 변수 참조 및 변경 가능.

---

1. **커링**

- 스칼라에서는 함수를 쉽게 커리할 수 있는 특수문법을 제공한다.
    - 커링 : 여러 인수를 받는 함수를 인수의 일부를 받는 여러 함수로 분할.
    - 따라서 스칼라에서는 커리된 함수를 직접 제공할 필요가 없다.

### 3.3 클래스와 트레이트

1. **간결성을 제공하는 스칼라의 클래스**

- 스칼라의 클래스와 인터페이스는 자바에 비해 더 유연함을 제공한다.
- 게터와 세터 :
    - 자바 : 필드마다 게터와 세터를 각각 만들어줘야 하기 때문에 클래스가 너무 복잡.
    - 스칼라 : 생성자, 게터, 세터가 암시적으로 생성되므로 코드가 훨씬 단순.

```java
// 자바의 게터와 세터
public class Student {
	private String name;
	private int id;

	public Student(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
}

// 스칼라의 게터와 세터
class Student(var name: String, var id: Int) // 암시적으로 생성자,게터,세터가 생성된 상태
```

---

1. **스칼라 트레이트와 자바 인터페이스**

- 스칼라는 트레이트라는 유용한 추상 기능( 자바의 인터페이스 대체 )을 제공한다.
- 트레이트 :
    - 추상 메서드와 기본 구현 메서드 모두 정의 가능.
    - 다중 상속 지원.
    - 상태 상속 가능.
    - 인스턴스화 과정에서도 조합 가능.

```java
// 스칼라 트레이트 정의
trait Sized {
	var size : Int = 0
	def isEmpty() = size == 0
}

// 트레이트 상속
class Empty extends Sized
new Empty().isEmpty() // true

// 트레이트 조합(인스턴스화 과정에서)
class Box
val box = new Box() with Sized // 인스턴스화 과정에서 트레이트 상속
box.isEmpty() // true
```

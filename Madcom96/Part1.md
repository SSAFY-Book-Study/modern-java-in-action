# Chapter 1. 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?
### 자바 8 이전의 변화
- 자바 5에서는 스레드 풀, 병렬 실행 컬렉션 등의 아주 강력한 도구를 도입했다
- 자바 7에서는 병렬 실행에 도움을 줄 수 있는 포크/ 조인 프레잌워크를 제공했다.
- 하지만 여전히 개발자가 활용하기는 쉽지 않았다.

### 자바 8에서 제공하는 새로운 기술
> *간결한 코드, 멀티코어 프로세서의 쉬운 활용이라는 요구사항이 기반이다.*
*이런 자바 8 기법은 함수형 프로그래밍에서 위력을 발휘한다.*
- 스트림 API
- 메서드 코드를 전달하는 기법
- 인터페이스의 디폴트 메서드

### 스트림 처리 + 동작 파라미터화
- 스트림이란 한 번에 한 개씩만들어지는 연속적엔 데이터 항목들의 모임
	- for-each 루프 등을 이용해서 외부 반복 할 필요 없이 내부 반복을 해 준다.
	- 필터링, 추출, 그룹화 등의 동작을 할 수 있다.
	- `List<Apple> heavyApples = inventory.stream().filter((Apple a) -> a.getWeight() > 150).collect(toList());`
	- parallelStream()을 대신 사용하면 병렬성을 얻을 수 있다.
- 동작 파라미터화란 코드 일부를 API로 전달하는 기능
	- 말 그대로 '동작'을 '파라미터'화 하는것.
- 이를 잘 이용하면 "병렬성을 공짜로 얻을 수 있다."

### 메서드와 람다를 일급 시민으로 사용하는 방법
1. 메서드 참조
	- `File[] hiddenFiles = new File(".").listFiles(File::isHidden)`
	- :: 는 이 메서드를 값으로 사용하라 는 의미
	- predicate 등의 자리에 사용 할 수 있다.
2. 람다 : 익명함수
	- `(Apple a) -> a.getWeigh() > 150`
	- 이름이 없는 함수이고 파라미터 등으로 전달 할 수 있다.
### 디폴트 메서드
- 하나의 함수만을 가지는 인터페이스를 선언
- 인터페이스를 구현하는 인스턴스들을 인터페이스로 묶어 해당 메서드를 사용 할 수 있다. 



---
# Chapter 2. 동작 파라미터화 코드 전달하기
- 변화하는 여러가지의 요구사항에 대응할 수  있다.
- boolean을 반환하는 함수를 predicate라고 한다. 선택조건을 결정할 수 있다.
	- `ApplePredicate`라는 인터페이스에 `test`라는 predicate 메서드를 만든다.
	- 이후 인터페이스를 상속하여 원하는 조건등에 맞게 `AppleGreenColorPredicate` 등을 구현해 줄 수 있다.
```java
// p로는 predicate를 받는다.
List<Apple> result = new ArrayList<>();
for (Apple apple: inventory){
	if (p.test(apple)) result.add(apple);
}
```
- 구현 조건은 인터페이스를 상속하여 원하는 조건에 맞게 구현하기 나름이 된다!
<br>

- 위와 같은 코드를 `filterApples`와 같이 메서드로 만들 때, predicate 자리에 람다 함수가 들어갈 수 있다.
	- 당연하지만 boolean을 반환하는 predicate의 모양을 맞춰주어야 한다.
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

---
# Chapter 3. 람다 표현식
### 형식
파라미터 리스트, 바디, 반환형식, 발생할 수 있는 예외 리스트 등을 가질 수 있다

### 특징
- 익명 : 이름이 없다.
- 함수 : 클래스에 종속되지 않아 함수라고 부른다.
- 전달 : 메서드 인수로 전달하거나 **변수로 저장할 수 있다**
- 간결성 : 익명클래스등 보다 간결하게 구현할 수 있다.

```java
(Integer i) -> {return "Alan" + i};
() -> "Alan"
// (파라미터) -> 바디({}로 묶어 줄 수도 있다.)
```
- 흐름 제어문이 들어갈 땐 {}로 바디를 묶어주어야 한다.
- 구문이 아닌 표현식이 바디에 들어갈 땐 return을 명시해 주어야 한다.

### 함수형 인터페이스
- 람다 표현식은 함수형 인터페이스 문맥에서 사용할 수 있다.
<br>

- 종류

**Predicate**
`T -> boolean`

**Consumer**
`T -> void`

**Supplier**
`() -> T`

**Function**
`T -> R`
*입력을 출력으로 매핑하는 람다를 정의할 때 사용할 수 있다.*

**Comparator**
`(T, T) -> int`

**Runnable**
`() -> void`

**Callable**
`() -> V`

### 형식 검사, 형식 추론, 제약
- 람다가 사용되는 콘텍스트를 이용해서 람다의 형식을 추론할 수 있다.
	- 대상 형식 : 어떤 콘텍스트에서 기대되는 람다 표현식의 형식
- 대상 형식을 이용해 람다 표현식과 함수 디스크럽터를 비교한다.
ex)
1. filter 메서드의 선언을 확인
2. filter 메서드는 두 번째 파라미터로 Predicate<Apple> 형식을 대상 형식으로써 기대한다.
3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크럽터를 묘사한다.
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야한다.


### 함수형 인터페이스가 람다표현식을 저장하게 할 수 있다.
```java
callable<Integer> c = () -> 42;
previlegedAction<Integer> p = () -> 42;
```

### 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡쳐할 수 있다.
- 람다 표현식이 지역변수를 사용하고 싶을 때에는 final로 상수로 설정된 변수, 혹은 실질적으로 상수로 사용되는 변수만 사용할 수 있다.
```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```
위와 같은 코드는 컴파일 되지 않는다.

### 클로저
- 클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스
- 다른 함수의 인수로 전달 할 수 있다.
- 클로저는 클로저 외부에 정의된 변수의 값에 접근하고, 값을 바꿀 수 있다.

### 메서드 참조
- 람다 표현식보다 메서드 참조가 더 가독성이 좋을 때도 있다.

```java
// 람다 표현식
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
// 메서드 참조
inventory.sort(comparing(Apple:getWeight));
```

**종류**
1. 정적 메서드 참조
2. 다양한 형식의 인스턴스 메서드 참조
3. 기존 객체의 인스턴스 메서드 참조
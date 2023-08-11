## Chapter2 동작 파라미터화 코드 전달하기

### 동작 파라미터화
  - 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블럭

왜 사용되는가? 왜 필요했을까?

- 현실 세계의 요구사항을 반영하기 위해서는 `변화에 대응할 수 있는 코드`를 작성해야 했기 때문

예를 들어 다음과 같은 상황에서 동적 파라미터화를 적용하면 요구사항을 만족시킬 수 있게 됨

- 리스트의 모든 요소에 대해서 `어떤 동작`을 수행할 수 있음
- 리스트 관련 작업을 끝낸 다음 `어떤 다른 동작`을 수행할 수 있음
- 에러가 발생하면 `정해진 어떤 다른 동작`을 수행할 수 있음
여기서 말하는 `어떤 동작`은 모든 요소에 대해서 `다양한 동작`을 의미하는 것 같다

### 변화하는 요구사항에 대응하기

기능을 구현하면서 `변화하는 요구사항`에 대응하기 위해 코드를 리팩토링하며 `유연한` 코드를 만드는 사례를 변화하는 요구사항에 따라 보여주고 있다
실제로 코딩테스트나 알고리즘 문제를 풀 때 많이 하는 고민인 것 같다

```
농장 재고목록 애플리케이션에 변화하는 요구사항 대응하기
```

1. 녹색 사과 필터링
재고 목록 관리 중 녹색 사과를 필터링 해야하는 기능이 요구되었다면 전체 사과 중 녹색을 가진 사과만 리스트에 담아 반환할 것이다
```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
  for(Apple apple : inventory) {
    if(GREEN.equals(apple.getColor()) result.add
  }
  return result;
```

당장에 필요한 요구사항은 수용되겠지만 녹색 사과만 필터링을 원했다면 빨간 사과를 필터링하는 요구사항이 생길지 모른다

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color) {
  for(Apple apple : inventory) {
    if(apple.getColor().equals(color)) result.add
  }
  return result;
```
위와 같이 수정하면 색에 대한 어떤 요구사항이 와도 enum에 색을 추가, 삭제만 해주면 대응할 수 있게 된다!

2. 무거운 사과 필터링

 ```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
  for(Apple apple : inventory) {
    if(apple.getWeight() > weight) result.add
  }
  return result;
```
위와 같이 무게를 입력으로 받아서 사과를 쉽게 필터링할 수 있다
하지만 색을 필터링하는 코드와 매우 유사하여 `DRY(don't repeat yourself)`원칙을 어기는 것과 같다고 한다

그렇다면 weight도 파라미터로 받고, flag를 두어서 하나의 메서드에서 색을 필터링, 무게를 필터링 하도록 할 수 있다
 ```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, Color color, int weight, boolean flag) {
  for(Apple apple : inventory) {
    ...
  }
  return result;
```
이러한 코드를 `형편없는 코드`라고 설명하고 있다
- flag의 의미를 전혀 알 수 없으며 요구사항이 바뀌었을 때 유연하게 대응할 수 없다

#### 이러한 상황에 대응하기 위해 동작 파라미터화를 이용하게 된 것이다

위에서 알아본 상황은 모두 Boolean으로 조건을 확인하였다
이에 유연하게 대응하기 위해 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 하며 **선택 조건을 결정하는 인터페이스**를 정의해서 사용한다

**프레디케이트를 통해 동작 파라미터화**

```java
public interface ApplePredicate {
  boolean test (Apple appl)
}

public class AppleWeightPredicate implements ApplePredicate {
...

public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  for(Apple apple : inventory) {
    if(p.test(apple)) result.add
  }
  return result;
```
**전달한 ApplePredicate 객체에 의해 filterApples의 동작이 결정된다**
변화에 유연하고 대응할 수 있는 코드가 준비되었으며 캡슐화가 이루어졌다고 볼 수 있다

#### 이제 우리는 한 개의 파라미터를 통해 다양한 동작에 대응할 수 있도록 준비가 되었다
그렇다면 다양한 동작을 어떻게 정의해서 파라미터로 던져주는게 좋은가?
한 곳에서만 사용할건데 위의 코드와 같이 전부 interface화 하여 `전략 패턴`을 통해 다양한 동작을 하도록 한다?
프로젝트 클래스 구조가 매우 복잡해질 위험이 있다

1. 익명 클래스 사용
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
  public boolean test(Apple, apple) {
    return RED.equals(apple.getColor());
  }
}
```
익명 클래스를 통해 클래스 선언과 인스턴스화를 동시에 할 수 있다
하지만 코드가 길고, 복잡해질수록 코드의 `장황함`이 발생하며 유지보수할 때 어딘가에 정의되어있을 이러한 익명 클래스를 관리하는 것은 비용이 클 수 밖에 없다

2. 람다 표현식 사용
```java
List<Apple> redApples = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
복잡하지 않은 인터페이스를 간단하고 명확하게 표현할 수 있게 되었으며 여러 상황에 대응할 수 있게 되었다

더 확장을 해나가기 위해 **형식을 추상화**할 수 있는데, 평생 사과만 관리하는 애플리케이션이 아닐 수 있다 다른 과일을 관리하는 애플리케이션이 되어도
같은 흐름의 필터링을 사용한다면 **형식을 추상화하면 된다**

### 유연성과 간결함을 모두 잡은 코드

```java
public interface Predicate<T> {
  boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  ...
  if(p.test(e)) {
  ...
}
```
### 정리
- `동작 파라미터화`에서는 메서드 내부적으로 `다양한 동작`을 수행할 수 있도록 `코드를 메서드 인수`로 전달한다
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있고, 유지보수 비용을 줄일 수 있다
- `코드 전달 기법`을 이용하면 **인터페이스를 상속받아 여러 클래스를 구현해야하는 수고를 없앨 수 있다**
- 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 `파라미터화`할 수 있다
---
## Chapter3 람다 표현식

Chapter2에서 변화하는 요구사항에 대응하기 위해 **동작 파라미터화**를 정리하였고, 그 결과 익명 클래스, 람다 표현식을 통해 다양한 동작을 구현할 수 있었다
특히 **람다 표현식**을 통해 깔끔하게 표현할 수 있었는데 이번 Chapter에서 람다 표현식에 대해 더 자세히 알아본다

### 람다란 ?
```
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있음
람다 표현식에는 이름은 없지만 `파라미터 리스트`, `바디`, `변환 형식`, `발생할 수 있는 예외 리스트`는 가질 수 있다
```
람다의 특징?
- 익명
  - 이름이 없으므로 **익명**이라 표현, 구현해야할 코드에 대한 걱정거리가 줄어듦?
- 함수
  - 람다는 메서드처럼 **특정 클래스에 종속되지 않으므로** 함수라고 부른다
  - 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다  
- 전달
  - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다
- 간결성
  - 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다
 
'''java
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
'''
- 파라미터 리스트
  - Comparator의 compare 메서드 파라미터(사과 두 개)
- 화살표
  - 화살표(->)는 람다의 파라미터 리스트와 바디를 구분
- 람다 바디
  - 두 사과의 무게를 비교, 람다의 반환값에 해당하는 표현식
 
|사용 사례|람다 예제|설명|
|---|---|---|
|불리언 표현식|```(List<String> list) -> list.isEmpty()```|반환값이 참, 거짓이다|
|객체 생성|```() -> new Apple(10)```|객체를 생성해서 반환|
|객체에서 소비|```(Apple a) -> { System.out.println(a.getWeight());}```||
|객체에서 선택/추출|```(String s) -> s.length()```|길이 반환|
|두 값을 조합|```(int a, int b) -> a * b```|결과 반환|
|두 값을 비교|```(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())```|a1이 크면 양수, a2가 크면 음수|
   


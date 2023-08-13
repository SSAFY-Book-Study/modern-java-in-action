# Part1

## 📔 CHAPTER1 - 자바 8, 9, 10, 11 : 무슨일이 일어나고 있는가?

책의 전반적인 밑거름을 제공

## 📔 CHAPTER2 - 동작 파라미터화 코드 전달하기

시시각각 변하는 사용자 요구사항에 대응하기 위해서 필요한 것들

1. 엔지니어링적인 비용 최소화
2. 새로 추가한 기능을 쉽게 구현
3. 장기적인 관점에서 유지보수가 쉬워야함

### 🔎 동작 파라미터화란?

**아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록**

- 기존의 방식대로 변화하는 요구사항에 대응하는 경우 중복되는 코드는 소프트웨어공학의 DRY(don't repeat yourself) 원칙을 어기는 방식이 된다.
- 따라서 **동작 파라미터화** 즉 메서드가 다양한 동작(또는 전략)을 받아서 내부적으로 다양한 동작을 수행할수 있는 방식을 선택해야 한다.
- 익명 클래스 &rarr; **람다 표현식**을 통해 유연성과 간결함을 가져야 한다.

## 📔 CHAPTER3 - 람다 표현식

### 🔎 람다 표현식란?

**람다 표현식이란 메서드로 전달할 수 있는 익명 함수를 단순화 한 것**

- 이름은 없지만 파라미터 리스트, 바디, 반환 형식, 예외 등은 가질 수 있다.
- 동작 파라미터를 이용할 때 처럼 판에 박힌 코드를 구현할 필요가 없다.
- 람다 표현식은 세 부분으로 이루어진다

  1. 파라미터  
     메서드 파라미터
  1. 화살표  
     화살표는 람다의 파라미터 리스트와 바디를 구분
  1. 바디  
     람다의 반환값에 해당하는 표현식

  **함수형 표현식**
  |함수형 인터페이스|함수 디스크립터|
  |------|---|
  |`Predicate<T>`|`T -> boolean`|
  |`Consumer<T>`|`T -> void`|
  |`Function<T, R>`|`T -> R`|
  |`Supplier<T>`|`() -> T`|

**컴파일러가 람다를 확인하는 방식**
람다가 사용되는 콘텍스트를 이용해서 **대상 형식**(람다 표현식의 형식)을 추론할 수 있다.

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

1. filter 메서드의 선언을 확인한다.
2. filter 메서드는 두번째 파라미터로 `Predicate<Apple>` 대상 형식을 기대한다.
3. `Predicate<Apple>`은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. test 메서드는 `Apple`를 받아. `boolean`을 반환하는 함수 디스크립터를 묘사한다.
5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야한다.

**람다 캡처링**  
람다 표현식에서는 **자유 변수**를 활용할 수 있다.
단, 지역 변수의 경우 final로 선언되어 있거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

> 왜 why?  
> 인스턴스 변수의 경우 힙에 저장되지만 지역 변수의 경우 스택에 위치한다. 람다가 지역 변수에 접근가능하다는 가정하에 람다가 스레드에서 실행된다면 스레드가 사라져 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서 해당 변수에 접근하려 할 수 있다.  
> -> 따라서 자유 지역 변수의 복사본을 제공되며, 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 할당해야 한다는 제약이 생긴것이다.

**메서드 참조**  
메서드를 호출하라고 명령을 하는 경우 설명을 참조하기 보단 직접 참조하는것이 편리하므로 명시적으로 메서드명을 참조함으로써 **가독성을 높일 수 있다**.
메서드 참조를 만드는 방법 3가지

1. 정적 메소드 참조
   Integer의 parseInt 메서드는 `Integer::parsInt`로 표현 가능
2. 인스턴스 메소드 참조
   String의 length 메서드는 `String::length`로 표현 가능
3. 기존 객체의 인스턴스 메서드 참조
   Transcation 객체를 할당받은 trasaction의 지역 변수의 경우 `transcation::getValue`로 표현 가능

**생성자 참조**  
`ClassName::new` 방식처럼 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.

**람다, 메서드 참조 활용하기**

1단계 : 코드전달

```java
public class AppleComparator implements Comparator<Apple> {
  public int compare(Apple a1, Apple a2){
    return a1.getWeight().compareTo(a2.getWeight);
  }
}
inventory.sort(new AppleComparator);
```

2단계 : 익명 클래스 사용

```java
inventory.sort(new AppleComparator(){
    public int compare(Apple a1, Apple a2){
    return a1.getWeight().compareTo(a2.getWeight);
  }
});
```

3단계 : 람다 표현식 사용

```java
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight));
```

4단계 : 메서드 참조 사용

```java
inventory.sort(comparing(Apple.getWeight));
```

**람다 표현식을 조합할 수 있는 유용한 메서드**

1. Comparator 조합

- Function 기반의 Comparator : `Comparator<Apple> c = Comparator.comparing(Apple.getWeight);`
- 역정렬 : `inventory.sort(comparing(Apple.getWeight), reversed());`

```java
inventory.sort(comparing(Apple.getWeight))
          .reversed()
          .thanComparing(Apple.getCountry);
```

2. Predicate 조합.

- negate, and, or 메서드를 제공.

```java
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(apple-> apple.getWeight() > 150)
                                                    .or(apple -> GREEN.equals(a.getColor()));
```

3. Function 조합

- andThen, compse 두가지 디폴트 메서드 제공

```java
Fuction<Integer, Integer> f = x -> x + 1;
Fuction<Integer, Integer> g = x -> x * 2;
Fuction<Integer, Integer> h1 = f.andThen(g); // h1.apply(1) : (1 + 1) * 2 => 4을 반환
Fuction<Integer, Integer> h2 = f.compose(g); // h2.apply(1) : (1 * 2) + 1 => 3을 반환
```

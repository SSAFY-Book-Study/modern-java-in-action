# Part 1. 기초

## Chapter 1 - 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?
  
 &nbsp;

> 자바 8에서는 새로운 기능인 스트림 API, 메서드에 코드를 전달하는 기법, 인터페이스의 디폴트 메서드 등의 기능을 추가하였다.

&nbsp;
- 스트림 처리 : 스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목의 모임, 자바 8에서는 우리가 하려는 작업을 고수준으로 추상화해서 일련의 스트림으로 만들어 처리하게 된다.
 
&nbsp;
 
 - 동작 파라미터화 : 코드 일부를 API로 전달하는 기능, 메서드 등이 포함된다.

&nbsp;

  ### 자바 함수

&nbsp;

> 프로그래밍 언어에서 함수라는 용어는 메서드, 특히 정적 메서드와 같은 의미로 사용된다.

 &nbsp;

자바 8에서는 함수를 새로운 값의 형식으로 추가했다. 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 만들었기 때문이다.
 
 &nbsp;
- 메서드 참조
```java 
File[] hiddenFiles = new File(".").listFiles(new FileFilter(){
    public boolean accept(File file){
        return file.isHidden();
    }
})
```
 &nbsp;

해당 코드를 자바 8에서는 다음과 같이 구현할 수 있다.

```java 
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
 &nbsp;
해당 코드와 같이 isHidden이라는 함수는 준비되어 있으므로 자바 8의 메서드 참조(::)를 통해 listFiles에 직접 전달할 수 있다.
 

 &nbsp;

- 람다 : 익명 함수
> 자바 8에서는 메서드를 일급값으로 취급할 뿐 아니라 람다(또는 익명함수)를 포함하여 함수도 값으로 취급할 수 있다.
 
 &nbsp;

- 프레디케이트 : 인수로 값을 받아 true나 false를 반환하는 함수
 
 &nbsp;
### 스트림

> 스트림 API에서는 라이브러리 내부에서 모든 데이터가 처리된다. 이와 같은 반복을 ***내부 반복*** 이라고 한다. 스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심!
 
 &nbsp;

 ### 디폴트 메서드와 자바 모듈
 
 &nbsp;
 > 자바 8에서는 인터페이스를 ***쉽게 변경할 수 있도록*** 디폴트 메서드를 지원한다.
 
 &nbsp;

 디폴트 메서드를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장할 수 있다.
 
 &nbsp;

 예를 들어 자바 8에서는 List에 직접 sort 메서드를 호출할 수 있다. 이는 자바 8의 List 인터페이스에 다음과 같은 디폴트 메서드 정의가 추가되었기 때문이다.
 
 &nbsp;
 ```java 
 default void sort(Comparator<? super E> c){
     Collections.sort(this.c);
 }
 ```
 
 &nbsp;
## Chapter 2 - 동작 파라미터화 코드 전달하기

 &nbsp;
 > 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 동작 파라미터화란 ***아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록***을 의미한다.

 &nbsp;
 ### 동작 파라미터화

 &nbsp;
 > 하나의 클래스의 속성과 관련된 모든 변화에 대응할 수 있는 유연한 코드를 작성할 수 있다. 예를 들면 컬렉션 탐색 로직과 각 항목의 적용할 동작을 분리할 수 있다는 강점이 있다.
 
 &nbsp;
 - 익명 클래스 : 자바의 지역 클래스와 비슷한 개념으로, 이름이 없는 클래스이다. 익명 클래스를 활용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다.
 
 &nbsp;
 > 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다. 변화하는 요구 사항에 더 잘 대응할 수 있는 코드를 구현할 수 있다.

  &nbsp;
  ## Chapter 3 - 람다 표현식
 
 &nbsp;
  > 람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

   &nbsp;
   ```java 
   // before
   Comparator<Apple> byWeight = new Comparator<Apple>(){
       public int compare(Apple a1, Apple a2){
           return a1.getWeight().compareTo(a2.getWeight());
       }
   };

   // after - 람다 표현식 적용 후
   Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
   ```

&nbsp;
람다 표현식은 파라미터, 화살표, 바디로 이루어진다.
 
 &nbsp;
 
 ### 어디에, 어떻게 람다를 사용할까?

  &nbsp;
  > 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

   &nbsp;

### 람다 활용  :실행 어라운드 패턴

&nbsp;

다음과 같은 형식의 코드를 실행 어라운드 패턴이라고 부른다.

 &nbsp;
 ```java 
 public String processFile() throws IOException {
     try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
         return br.readLine();
     }
 }
 ```

&nbsp;

- 1단계 : 동작 파라미터화를 기억
 
 &nbsp;
- 2단계 : 함수형 인터페이스를 이용해서 동작 전달

 &nbsp;
- 3단계 : 동작 실행 

 &nbsp;
 - 4단계 : 람다 전달
 
 &nbsp;
 ### 함수형 인터페이스 사용

  &nbsp;
  > 함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터라고 한다.

   &nbsp;
   - Predicate
   > java.util.function.Predicate<T> 인터페이스는 test라는 추상 메서드를 정의한다.

&nbsp;
- Consumer
> java.util.function.Consumer<T> 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.

 &nbsp;
- Function
> java.util.function.Function<T, R> 인터페이스는 제네릭 형식 T를 인수로 받아서 제니릭 형식 R 객체를 반환하는 추상 메서등 apply를 정의한다.

 &nbsp;
 ### 형식 검사, 형식 추론 제약

  &nbsp;
> 람다가 사용되는 컨텍스트를 이용해서 람다의 형식을 추론할 수 있다.

 &nbsp;
 자바 컴파일러는 람다 표현식이 사용된 컨텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

  &nbsp;
### 클로저
 &nbsp;
  > 클로저란 함수의 비지역 변를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다.

   &nbsp;
### 메서드 참조

&nbsp;

> 메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다. 설명을 참조하기 보다는 메서드명을 직접 참조하는 것이다. 예를 들면 다음과 같다.

&nbsp;
```java 
// before
(Apple apple) -> apple.getWeight()

// after
Apple::getWeight
```

&nbsp;
### 람다, 메서드 참조 활용하기

&nbsp;
코드의 흐름으로 살펴보자.
```java 
// 1. 코드 전달
void sort(Comparator<? super E> c);

public class AppleComparator implements Comparator<Apple>{
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());

// sort의 동작은 파라미터화 되었다고 표현할 수 있다.

// 2단계 : 익명 클래스 사용

inventory.sort(new Comparator<Apple>(){
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeigh());
    }
})

// 3단계 : 람다 표현식 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 콘텍스트를 활용해 람다의 파라미터 형식을 추론!
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 4단계 : 메서드 참조
import java.util.Comparator.comparing;

inventory.sort(comparing(Apple::getWeight));

// 아래로 지나면서 코드가 간소화 됨과 동시에 의도 또한 명확히 보여지는 것을 알 수 있다!
```


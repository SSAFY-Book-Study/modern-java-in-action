## Chapter4 스트림 소개

### 스트림이란?

스트림이 없던 자바에서는 컬렉션에 대한 단순 반복 처리 코드가 복잡하며 성능을 올리기 위한 병렬 처리 코드는 더 복잡하다

스트림은 컬렉션 반복을 멋지게 처리하는 기능이라고 생각
**스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다**

자바7에서의 컬렉션 처리
```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu) {    // <- 누적자로 요소 필터링
  if(dish.getCalories() < 400) {
    lowCaloricDishes.add(dish);
  }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish>() {  // <- 익명 클래스로 요리 정렬
  public int compare(Dish dish1, Dish dish2) {
    return Integer.compare(dish1.getCalories(), dish2.getCalories());
  });
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) {
  lowCaloricDishesName.add(dish.getName());  // <- 정렬된 리스트를 처리하면서 요리 이름 선택
}
```
lowCaloricDishes라는 `가비지 변수`를 사용했으며 요구사항에 따라 반복자를 통해 처리를 하는 복잡하고 긴 코드이며 유지보수 비용이 크다

자바8에서의 컬렉션 처리
```java
List<String> lowCaloricDishesName =
            menu.stream()
            .filter(d -> d.getCalories() < 400)    // <- 400칼로리 이하 요리 선택
            .sorted(comparing(Dish::getCalories))  // <- 칼로리로 요리 정렬
            .map(Dish::getName)    // <- 요리명 추출
            .collect(toList());    // <- 모든 요리명을 리스트에 저장
```
stream을 사용하면 변화하는 요구사항에도 쉽게 대응할 수 있으며 Stream을 parallelStream으로 변경하면 병렬로 처리하는 성능을 올릴 수 있다

자바8 스트림 API의 특징
- 선언형: 더 간결하고 가독성이 좋아짐
- 조립할 수 있음: 유연성이 좋아진다
- 병렬화: 성능이 좋아진다

### 스트림 시작하기

스트림이란 `데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소`로 정의할 수 있음

- 연속된 요소
  - 컬렉션과 마찬가지로 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공
  - 컬렉션은 시간과 공간 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룸 (데이터)
  - 스트림은 filter, sorted, map처럼 표현 계산식이 주를 이룸  (계산)
- 소스
  - 데이터 제공 소스로부터 데이터를 소비
  - 전달받은 데이터 순서 유지
- 데이터 처리 연산
  - 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원
 
스트림의 중요 특징   
- 파이프라이닝
  - 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 **스트림 자신을 반환**
  - `게으름`, `쇼트서킷`같은 최적화
- 내부 반복
  - 반복자를 통한 명시적 반복인 외부 반복과 달리 내부 반복을 지원
  - 반복을 처리하고 결과 스트림 값을 어딘가에 저장해줌

### 스트림과 컬렉션

스트림과 컬렉션의 차이를 설명하고 있다
비유를 들어 설명하고 있는데
컬렉션은 **DVD에 저장된 영화로 모든 데이터를 내려받고 가지고 있다**고 표현
스트림은 **인터넷으로 스트리밍하는 영화로 모든 데이터를 내려받지 않고 필요할 때 값을 받아 처리**한다고 표현하고 있다
- 데이터를 언제 계산하느냐?
- 모든 값을 저장, 처리해야 하는가?
- 반복 처리 방식 (외부 반복과 내부 반복)

### 스트림 연산

앞서 살펴보았을 때 스트림은 **파이프라이닝을 통해 스트림 연산끼리 연결하여 결과로 스트림을 반환**한다고 하였다
이에 따라 **연결할 수 있는 스트림 연산을** `중간 연산`이라고 하며 **스트림을 닫는 연산을** `최종 연산`이라고 한다

중간 연산
- filter나 sorted 같은 중간 연산은 다른 스트림을 반환
- **단말 연산을 스트림 파이프라인에 실행하기 전**까지는 아무 연산도 수행하지 않는다는 것 -> **게으르다**
  - 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리
 
최종 연산
- 최종 연산에 의해 스트림 파이프라인에서 결과를 도출
  - 도출한 결과는 List, Integer, void 등 스트림 이외의 결과가 반환
 
스트림 이용 과정은 세 가지로 요약할 수 있음
1. 질의를 수행할 (컬렉션 같은) 데이터 소스
2. 스트림 파이프라인을 구성할 중간 연산 연결
3. 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

### 정리

- 스트림은 소스에서 추출된 연속 요소로 데이터 처리 연산을 지원
- 스트림은 내부 반복을 지원하며 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화
- 스트림에는 중간 연산과 최종 연산이 있음
- 중간 연산은 filter, map 처럼 스트림을 반환하면서 다른 연산과 연결되는 연산
- forEach나 count 처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 한다
- 스트림의 요소는 요청할 때 게으르게 계산된다

## Chapter5 스트림 활용

5장에서는 스트림의 다양한 연산을 설명한다

### 필터링
스트림 인터페이스는 filter 메서드를 지원
**프레디케이트(불리언을 반환하는 함수)**를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환
```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish:isVegetarian)
                                .collect(toList());
```
#### 프레디케이트를 이용한 슬라이싱

`TAKEWHILE`을 활용하면 해당 조건을 만족하지 않는 요소를 만났을 때 종료시킬 수 있다
예를들어 칼로리 순으로 정렬된 `specialMenu`를 예로 들어본다
```java
List<Dish> slicedMenu = specialMenu.stream()
                                .filter(dish -> dish.getCalories() < 320)
                                .collect(toList());
```
위의 코드는 모든 요소를 반복한다 리스트가 정렬되어있음을 이용하면 320보다 크거나 같은 값을 만났을 때 더 탐색하지않아도 된다
`takeWhile`을 이용하면 스트림을 슬라이스 할 수 있다
```java
List<Dish> slicedMenu = specialMenu.stream()
                                .takeWhile(dish -> dish.getCalories() < 320)
                                .collect(toList());
```
320칼로리보다 큰 요소를 선택하고 싶다면 `dropWhile`을 이용할 수도 있다 
```java
List<Dish> slicedMenu = specialMenu.stream()
                                .dropWhile(dish -> dish.getCalories() < 320)
                                .collect(toList());
```
320칼로리 미만의 음식은 버린다

#### 스트림 축소
마찬가지로 모든 결과를 요구하지 않는 경우도 있기 때문에 전체를 탐색하지 않고 최대 요소 n개를 반환하도록 할 수 있다
```java
List<Dish> slicedMenu = specialMenu.stream()
                                .filter(dish -> dish.getCalories() < 320)
                                .limit(3)
                                .collect(toList());
```
요소를 탐색하는데 순차적으로 탐색하지 않고 건너 뛸 수도 있다
```java
List<Dish> dishes = specialMenu.stream()
                                .filter(dish -> dish.getCalories() < 320)
                                .skip(2)
                                .collect(toList());
```

### 매핑
스트림의 각 요소에 함수를 적용시키는 연산을 수행할 수 있으며 스트림은 `map`메서드를 지원한다
```java
List<String> dishNames =  menu.stream()
                              .map(Dish::getName)
                              .collect(toList());
```
#### 스트림 평면화
"hello"와 "world"에서 알파벳을 추출하는데 중복을 제거하여 알파벳을 추출하고 싶을때 스트림을 이용해 추출해볼 수 있다
```java
List<String> words = List.of("hello", "world");
words.stream()
      .map(word -> word.split(""))
      .distinct()
      .collect(toList());
```

하지만 결과는 예상과 다르게 나타난다

![스크린샷 2023-08-19 오후 1 50 35](https://github.com/dpwns523/modern-java-in-action/assets/84260096/94b80507-b200-4f2f-9a79-5795c5f5cfc7)[모던 자바 인 액션 p163]

map의 결과가 Stream<String[]>으로 각 원소 "h", "e", "l", "l", "o"가 하나의 String[]이면서 Stream으로 감쌌다
그 이후 Distinct를 진행하니 각 Stream이 같지 않아 걸러지지 않고 우리가 기대한 결과를 반환하지 않는다

**flatMap이라는 메서드를 이용하면 이 문제를 해결할 수 있다**
```java
List<String> words = List.of("hello", "world");
words.stream()
      .map(word -> word.split(""))  // <- 각 단어를 개별 문자를 포함하는 배열로 변환
      .flatMap(Arrays::stream)  // <- 생성된 스트림을 하나의 스트림으로 평면화
      .distinct()
      .collect(toList());
```

![스크린샷 2023-08-19 오후 1 54 32](https://github.com/dpwns523/modern-java-in-action/assets/84260096/b9d0170e-b008-4dea-8e2c-3dae4aaa804d)[모던 자바 인 액션 p165]

### 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 `allMatch`, `anyMatch`, `noneMatch`, `findFirst`, `findAny` 등 다양한 유틸리티 메서드를 제공
스트림 API의 장점은 메서드명을 보고 어떤 역할을 하는 지 알기 쉽다. 각 예시를 정리하진 않겠다

#### Optional
Optional<T>클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다 위에서 언급한 메서드는 결과를 반환할수도, 반환하지 않을 수 있다 이에 따라 null 에러를 발생시키기 쉽기 때문에 Optional<T>를 만들었다

### 리듀싱
스트림의 모든 요소를 반복적으로 처리하는 질의를 **리듀싱 연산**이라고 한다
요소의 합, 최댓값, 최솟값, 요소 수 등 전체 요소를 반복적으로 탐색하는 방식에서 리듀싱 연산이 사용될 수 있다
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
or
int sum = numbers.stream().reduce(0, Integer::sum);
```
그렇다면 일반적인 반복문을 사용하지 않고 스트림의 리듀싱 연산을 사용해야하는 이유가 있을까?

- 스트림이므로 내부 반복이다
  - 내부 반복이 추상화되면서 내부 구현에서 병렬로 `reduce`를 실행할 수 있게 된다 

### 숫자형 스트림
```java
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
or
int sum = numbers.stream().reduce(0, Integer::sum);
```
위에서 알아봤던 reduce방식에서 Integer를 int로 언박싱하는 박싱 비용이 숨어있다.

Stream<T>를 효율적으로 처리할 수 있도록 **기본형 특화 스트림**을 제공한다

`int` 요소에 특화된 `IntStream`, `double -> DoubleStream`, `long -> LongStream`을 제공

```java
int sum = numbers.stream()
                  .mapToInt(Integer::intValue)  // <- IntStream 반환
                  .sum();
```
IntStream은 `max`, `min`, `average`등 다양한 유틸리티 메서드도 지원한다

마찬가지로 IntStream을 다시 Stream으로 변환하기 위해 boxed 메서드를 지원하고 있다

IntStream을 이용하면 원하는 숫자 범위를 filtering하거나, 숫자 범위를 생성, 필요 연산을 통해 조건에 맞는 집합을 생성할 수 있다
이 부분은 현재 장에선 생략하도록 한다

### 정리
- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다
- filter, distinct, takeWhile, dropWhile, skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다
- 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메서드를 효과적으로 사용할 수 있다
- map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다
- findFirst, findAny 메서드로 스트림의 요소를 검색할 수 있다.
- allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다
- 쇼트서킷, 즉 결과를 찾는 즉시 반환하며 전체 스트림을 처리하지 않는다
- reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출
- filter, map 등은 상태를 저장하지 않는 **상태 없는 연산**, reduce같은 연산은 상태를 저장, sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 한다 -> **상태 있는 연산**

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


## Chapter6 스트림으로 데이터 수집
스트림에서 최종 연산 `collect`를 사용하는 방법을 확인했다
이번 장에서는 `Collector`인터페이스에 정의된 다양한 요소 누적 방식을 인수로 받아 스트림을 최종 결과로 도출하는 리듀싱 연산을 알아본다

### 컬렉터란 무엇인가?
스트림의 `collect`메서드로 `Collector`인터페이스 구현을 전달하여 스트림의 요소를 어떤 식으로 도출할지 지정한다 
- `Collectors` 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다
  - 기능
    - 스트림 요소를 하나의 값으로 리듀스하고 요약
    - 요소 그룹화
    - 요소 분할
          
`collect`로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의하는 방법을 알아본다

1. 메뉴에서 요리 수를 계산하는 counting() 팩토리 메서드
   ```java
        long howManyDishes = menu.stream().collect(Collectors.counting());
   ```
2. 스트림값에서 최댓값과 최솟값 검색
   ```java
         Comaprator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
         Optional<Dish> mostCalorieDish = menu.stream()
                                               .collect(maxBy(dishCaloriesComparator));  // 최솟값: minBy
   ```
3. 스트림 객체의 숫자 필드의 합계나 평균 등을 반환하는 **요약 연산**
     ```java
        int totalCalories = menu.stream()
                                 .collect(summingInt(Dish::getCalories));  // 합계, summingLong, summingDouble도 제공
        double avgCalories = menu.stream()
                                 .collect(averagingInt(Dish::getCalories));  // 평균, averagingLong, averagingDouble 제공

   ```
4. **요약 연산**으로 메뉴에 있는 `요소 수`, `요리의 칼로리 합계`, `평균`, `최댓값`, `최솟값` 등을 계산하는 `summarizingInt`
     ```java
        IntSummaryStatistics menuStatistics = menu.stream()
                                 .collect(summarizingInt(Dish::getCalories));
        // IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}

   ```
5. 문자열 연결
   ```java
        String shortMenu = menu.stream().collect(joining());  // 내부적으로 StringBuilder를 이용해 문자열을 하나로 만든다
   ```
위의 `Collector`인터페이스가 제공하는 정적 팩토리 메서드는 모두 `Collectors.reducing`으로도 구현가능하다
  ```java
        int totalCalories = menu.stream()
                                 .collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
  ```
reducing은 인수 세 개를 받는다
- 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값이다
- 칼로리를 정수로 변환할 때 사용한 변환 함수
- 두 항목을 하나의 값으로 더하는 BinaryOperator

한 개의 인수만 전달하는 방식도 있다
  ```java
        Optional<Dish> mostCalorieDish = menu.stream()
                                             .collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
  ```
- 첫 번째 요소를 시작 요소(첫 번쨰 인수가 됨)
- 자기 자신을 그대로 반환하는 **항등 함수** (두 번쨰 인수가 됨)
- 빈 스트림이 넘겨질 수 있어서 Optional을 반환

**같은 연산도 다양한 방식으로 수행할 수 있다**
가독성과 성능이라는 두 마리 토끼를 잡을 수 있는 최적의 해법을 선택하자


#### 그룹화
이어서 `Collectors`에서 제공하는 그룹화 **분류 함수**에 대해서 알아본다
  ```java
        Map<Dish.Type, List<Dish>> dishesByType = menu.stream()
                                             .collect(groupingBy(Dish::getType));
    /* 결과
        {FISH=[prawns, salmon], MEAT = [.. , ...], OTHER=[...]}
    /*
  ```
데이터에 필터링을 한 뒤 그룹화를 하고싶을 수 있고, 여러 기준으로 그룹화를 하고싶을 수 있다

#### filter 예제
  ```java
        Map<Dish.Type, List<Dish>> dishesByType = menu.stream()
                                                      .filter(dish -> dish.getCalories() > 500)
                                                      .collect(groupingBy(Dish::getType));
    /* 결과
        {OTHER=[french fries, pizza], MEAT=[pork, beef]}
        FISH 종류 요리는 없음을 나타내야하는데 filter가 먼저 적용되어 Key 자체가 사라지는 문제가 발생
    /*
        Map<Dish.Type, List<Dish>> dishesByType = menu.stream()
                                                      .collect(groupingBy(Dish::getType,
                                                                  filtering(dish -> dish.getCalories() > 500, toList()));
    /* 결과
        {OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}
        filtering 메서드는 Collectors 클래스의 프레디케이트 인수로 각 그룹의 요소와 필터링 된 요소를 재그룹화 
    /*
  ```
#### 다수준 그릅화 예제
MEAT 분류 중에서 Calory에 따라 분류를 하고싶다면 두 번째 기준을 정의하는 방식을 적용해볼 수 있다
  ```java
        Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
          menu.stream().collect(
              .groupingBy(Dish::getType,
                  groupingBy(dish -> {
                      if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                      else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                      else CaloricLevel.FAT;
                  })
              ));
    /* 결과
        {MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[prawns], NORMAL=[salmon]},
        OTHER ={...}, }
    /*
  ```
마찬가지로 한 번 그룹화를 한 후 반환된 컬렉터에서 다른 형식을 적용시킬 수 있다

다음 코드는 각 요리 종류별 최고 칼로리 음식을 맵으로 반환하는 예시이다
 ```java
        Map<Dish.Type, Dish> mostCaloricByType =
          menu.stream()
              .collect(groupingBy(Dish::getType,  // <- 분류 함수
                        collectingAndThen(
                            maxBy(comparingInt(Dish::getCalories)),  // <- 감싸인 컬렉터
                        Optional::get)));  // <- 변환 함수
    /* 결과
        {FISH=salmon, MEAT=pork, OTHER=pizza}
    /*
  ```
### 분할
**분할 함수**를 이용하면 맵의 키 형식을 `Boolean`으로 프레디케이트의 결과로 두 개의 그룹으로 나눈다음 그룹화할 수 있다
이전에 살펴보았던 `groupingBy`와 유사하지만 둘의 차이점은 `Function`으로 분리할지, `Predicate`로 분류할지에 대한 차이가 있는 것 같다.

채식인지 육식인지로 분할한 후 요리 종류로 그룹화하는 예시이다
 ```java
        Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream().
        collect(
            partitioningBy(Dish::isVegetarian,  // <- 분할 함수
                          groupingBy(Dish::getType)));  // <- 두 번째 컬렉터
    /* 결과
        {false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]},
         true={OTHER=[french fries, rice, season fruit, pizza]}}
    /*
  ```

### Collector 인터페이스
Collector 인터페이스의 다양한 컬렉터 메서드를 확인해봤는데, 더 효율적으로 문제를 해결하는 컬렉터를 직접 구현해볼 수 있다
```java
public interface Collector<T, A, R> {
  Supplier<A> supplier();
  BiConsumer<A, T> accmulator();
  Function<A, R> finisher();
  BinaryOperator<A> combiner();
  Set<Charateristics> characteristics();
}
```
- T는 수집될 스트림 항목의 제네릭 형식
- A는 누적자, 수집 과정에서 중간 결과를 누적하는 객체 형식
- R은 수집 연산 결과 객체의 형식

### 커스텀 컬렉터를 구현해서 성능 개선

#### supplier() - 빈 결과로 이루어진 Supplier 반환 
#### accmulator() - 누적자와 n번째 요소를 함수에 적용하여 컨테이너에 요소 추가
#### finisher() - 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환 (누적 과정을 끝낼 때 호출할 함수를 반환)
#### combiner() - 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이결과를 어떻게 처리할지 정의 (결과 컨테이너 병합)
#### characteristics() - 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
  - Characteristics는 스트림을 병렬로 리듀스할 것인지 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트 제공
    - UNORDERED: 어떤 순서로 방문해도 영향이 없다
    - CONCURRENT: 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있다 -> 병렬 리듀싱 수행이 가능하다 (UNORDERED를 함께 설정하지 않았다면 데이터 소스가 정렬되지 않은 상황에서만 병렬 리듀싱 가능)
    - IDENTITY_FINISH: 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다 -> 추가 형변환이 필요 없다
   
직접 컬렉터를 구현하면 적용시킬 연산에 최적화할 수 있다

### 정리
- collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법을 인수로 갖는 최종 연산이다
- 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최소, 최대, 평균을 계산하는 컬렉터 등이 미리 정의되어 있다
- groupingBy로 스트림 요소를 그룹화하거나, partitioningBy로 분할할 수 있다
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다



## Chapter7 병렬 데이터 처리와 성능

스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀 수 있다 
이번 장에서 내부적인 병렬 스트림 처리는 어떻게 이루어지는 지 알아본다

### 병렬 스트림
컬렉션에 parallelStream을 호출하면 **병렬 스트림**이 생성됨

병렬 스트림이란
- 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림

### 순차 스트림을 병렬 스트림으로 변환
순차 스트림에 parallel 메서드를 호출하면 함수형 리듀싱 연산이 병렬로 처리된다

```java
  public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
                  .limit(n)
                  .parallel()    // <- 병렬 스트림으로 변환
                  .reduce(0, Long::sum);
```
반대로 `sequentail`로 병렬 스트림을 순차 스트림으로 바꿀 수 있다
```java
  stream.parallel()  // 병렬
        .filter(...)
        .sequential()  // 순차
        .map(...)
        .parallel()    // 병렬
        .reduce();
```

**병렬 스트림을 활용하여 성능을 올리기전에 공유 데이터에 대해 상태를 바꾸는 연산이 있는지, 상황에 따라 성능 개선을 기대해야 한다**

### 병렬 스트림 효과적으로 사용하기
병렬 스트림 사용은 상황에 따라 성능이 달라지기 때문에 정해진 기준이 없어 고려해야할 사항들을 정리한다
- 순차 스트림을 병렬 스트림으로 쉽게 바꿀 수 있다 하지만 무조건 병렬 스트림이 빠른 것은 아니므로 직접 성능을 측정하는 것이 좋다
- 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소이므로 기본형 특화 스트림을 사용하는것이 좋다
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다
  - limit, findFirst와 같이 요소의 순서에 의존하는 연산
- 전체 파이프라인 연산 비용을 고려
  - 요소 수가 N, 처리 비용이 Q이면 스트림 파이프라인 처리 비용은 N * Q Q가 높아진다는 것은 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있음을 의미
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다
  - 병렬화 과정에서 생기는 부가 비용을 상쇄할 수 있을 만큼의 이득을 얻지 못함
- 스트림을 구성하는 자료구조가 적절한지 확인하라
  - ArrayList는 효율적으로 분할할 수 있지만 LinkedList는 분할하려면 모든 요소를 탐색해야 한다
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있음
- 최종 연산의 병합과정 비용을 살펴보라

### 포크/조인 프레임워크

포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 서브태스크 각각의 결과를 합쳐서 전체 결과를 만들도록 설계

서브태스크르 스레드 풀의 작업자 스레드에 분산 할당하는 `ExecutorService`인터페이스를 구현

![스크린샷 2023-08-20 오후 6 54 11](https://github.com/dpwns523/modern-java-in-action/assets/84260096/5c8a89ea-e618-4d6d-91bc-022404a6c924) [모던 자바 인 액션 p256]
```java
// RecursiveTask를 상속받아 포크조인 프레임워크에서 사용할 테스크 생성
public class ForkJoinSumCalculator extends RecursiveTask<Long> {  
        
    private final long[] numbers; // 더할 숫자 배열
    private final int start; // 이 서브테스크에서 처리할 배열의 초기 위치와 최종 위치
    private final int end;
    public static final long THRESHOLD = 10_000; // 이 값 이하의 서브테스크는 더 이상 분할 x


    // 메인 테스크를 생성할 때 사용할 공개 생성자
    public ForkJoinSumCalculator(long[] numbers) { 
        this(numbers, 0, numbers.length);
    }

    // 메인 테스트의 서브테스크를 재귀적으로 만들 때 사용할 private 생성자
    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start; // 이 테스크에서 더할 배열의 길이
        if (length <= THRESHOLD) {
            // 기준값과 같거나 작으면 순차적으로 결과 계산
            return computeSequentially();
        } 
        // 배열의 첫 번째 절반을 더하도록 서브테스크 생성
        ForkJoinSumCalculator leftTask = 
                new ForkJoinSumCalculator(numbers, start, start+length/2);
        // ForkJoinPool의 다른 스레드로 새로 생성한 테스크를 비동기로 실행
        leftTask.fork();
        // 배열의 나머지 절반 더하는 서브테스크 생성
        ForkJoinSumCalculator rightTask =
                new ForkJoinSumCalculator(numbers, start, start+length/2);
        // 두번째 서브테스크 동기 실행. 추가 분할 발생할 수 있음
        Long rightResult = rightTask.compute();
        // 첫번째 서브테스크 결과를 읽거나 아직 결과가 없으면 기다림
        Long leftResult = leftTask.join();
        // 두 서브테스크의 결과를 조합한 값이 이 테스크의 결과값
        return rightResult + leftResult; 
    }

    // 더 분할할 수 없을 때 서브테스크의 결과를 계산하는 알고리즘
    private Long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }
}
```
#### 포크/조인 프레임워크를 제대로 사용하는 방법
- join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시키므로 두 서브태스크가 모두 시작된 다음 join을 호출해야 함
  - 잘못 호출하면 각각의 서브태스크가 다른 태스크가 끝나길 기다리는 일이 발생
- RecursiveTask내에서는 ForkJoinPool의 invoke 메서드를 사용하지 말아야 한다
- 서브태스크에 fork를 호출해서 ForkJoinPool의 일정을 조율하는 것 보다 compute를 호출하는 것이 효율적이다
- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅이 어렵다
  - 포크/조인 프레임워크에서는 fork라 불리는 다른 스레드에서 compute를 호출하므로 스택 트레이스가 도움되지 않는다
  - 멀티코어에 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠르지는 않다
- 병렬 처리로 성능 개선하려면 테스크를 여러 독립적인 서브테스크로 분할할 수 있어야 한다
  - 각 서브테스크의 실행시간은 새로운 테스크를 포킹하는데 드는 시간보다 길어야한다
  - JOT 컴파일어에 의해 최적화되려면 몇 차례의 준비 과정 및 실행 과정을 거쳐야 한다
  - 따라서 성능 측정은 여러번 프로그램을 실행한 결과를 측정해야한다

### 작업 훔치기

코어 개수와 관계없이 적절한 크기로 분할된 많은 테스크를 포킹하는 것은 바람직하다

이론적으로는 코어 개수만큼 병렬화된 테스크로 작업부하를 분할하면 모든 CPI 코어에서 이를 실행할 것이고, 크기가 같은 각각의 테스크는 같은 시간에 종료될 것 같지만 현실에서는 작왑완료 시간이 제각각으로 달라질 수 있다

이를 작업 훔치기 기법으로 해결

- ForkJoinPool의 모든 스레드를 공정하게 분할한다
  - 각각의 스레드는 할당된 테스크를 포함하는 이중 연결 리스트를 참조하며 작업이 완료될 때마다 큐의 헤드에서 다른 테스크를 가져와 작업을 처리
- 한 스레드는 다른 스레드보다 할당된 테스크를 빨리 처리할 수 있다
  - 작업이 완료된 스레드는 유휴 상태로 바뀌는게 아닌 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다

따라서 테스크 크기를 작게 나누어야 작업자 스레드 간의 작업부하를 비슷한 수준으로 유지 가능

![스크린샷 2023-08-20 오후 7 34 27](https://github.com/dpwns523/modern-java-in-action/assets/84260096/37c11a66-c802-4dfe-8d45-ce9d6d9a3cdb)[모던 자바 인 액션 p262]

### Spliterator 인터페이스

Spliterator는 분할할 수 있는 반복자라는 의미로 병렬 작업에 특화

```java
public interface Spliterator<T> {
	boolean tryAdvance(Consumer<> super T> action);
	Spliterator<T> trySplit();
	long estimateSize();
	int characteristics();
}
```
- tryAdvance(Consumer<> super T> action) : 탐색해야 할 요소가 남아있는지?
- trySplit() : Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성
- estimateSize() : 탐색해야할 요소 수 정보
- characteristics() : Spliterator의 특성 집합 반환 (enum)

### 정리
- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리할 수 있음
- 간단하게 스트림을 병렬로 처리할 수 있지만 항상 빠른것은 아님
  - 병렬 처리 동작 방법과 성능은 직간적이지 않을 때가 많아 성능을 직접 측정해봐야 한다
- 데이터가 아주 많거나 각 요소를 처리하는 데 오랜 시간이 걸릴 때 성능을 높일 수 있음
- 가능한 기본형 특화 스트림을 사용하는 등 올바른 자료구조 선택이 병렬로 처리하는 것보다 성능적으로 더 큰 영향을 미칠 수 있음
- 포크/조인 프레임워크는 병렬화할 수 있는 태스크를 작은 태스크로 분할 -> 분할된 태스크를 각각의 스레드로 실행 -> 서브태스크 각각의 결과를 합쳐 최종 결과 도출
- Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의   

 


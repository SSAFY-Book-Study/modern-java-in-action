<h1>Part2. 함수형 데이터 처리</h1>
<br>

<h2> 1. 스트림 소개</h2>

> 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- `선언형`으로 코드를 구현함. <-> 명령형 <br>
  : 간결, 가독성이 좋아짐
- 복잡한 데이터 처리를 하나의 파이프라인으로 처리할 수 있음.<br>
  : 유연성, 성능이 좋아짐
- `데이터 소스의 스트림` + 여러 개의 `중간연산` + 하나의 `최종연산`으로 이루어짐.<br>

  - **중간 연산**

    - 반환값은 Stream으로 다른 중간연산의 입력 stream이 될 수 있음.(`스트림 요소 소비X`)<br>
    - `lazy한 특성`으로 인해 최종연산 전까지는 수행되지 않음.<br>

  - **최종 연산**
    - 반환 값이 stream이 아니기 때문에 한 번만 수행함.(`스트림 요소 소비O`)

<h3>컬렉션 vs 스트림</h3>

|                      |         컬렉션         |      스트림      |
| :------------------: | :--------------------: | :--------------: |
|         주제         |         데이터         |       계산       |
|         반복         |        외부반복        |     내부반복     |
| 데이터 <br>처리 시기 | 모든 값을<br>처리한 후 | 요청마다<br>처리 |
|    반복 사용<br>     |           O            |        X         |

<h2> 2. 스트림 활용</h2>

이 챕터의 내용은 `스트림 메소드의 종류`에 대해 서술하기에 퀴즈를 풀면서 나의 보완점과 주석으로 공부하려고 한다.<br>

<h4>퀴즈</h4>

1. **2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.<br>**

```java
Optional<Transaction> TransactionsIn2011 = transactions.stream()
                                                        .filter(transaction -> transaction.getYear==2011)
                                                        .sorted(comparing( Transaction::getValue))
                                                        .collect(toList());

```

- comparing은 뭘까?<br>
  sorted의 매개변수로는 comparator<T>, 즉 정렬기준을 넣어줘야한다. 이때 Comparator의 디폴트 메소드 `comparing()는 Fuction<T,R>을 인자로 받고 Compartaor를 반환`한다. comparing의 매개변수로 정렬 기준을 넣어주면 그 기준대로 정렬된다. <br>

  > 다수 기준으로 정렬을 원할경우 comparing() 다음 thenComparing()을 사용하면 된다.<br>

  **_값, 이름순으로 정렬하기 위해서는 sort 매개변수에 comapring()하고 thenComparing()하면 된다._**

  ```java
  Optional<List<Transaction>> TransactionsIn2011 = transactions.stream() // Stream<Transaction>
                                                        .filter(transaction -> transaction.getYear==2011) // Stream<Transaction>
                                                        .sorted(comparing( Transaction::getValue).thenComparing(Transaction::getName)) // Stream<Transaction>
                                                        .collect(toList()); // List<Transaction>

  ```

2. **거래자가 근무하는 모든 도시를 중복 없이 나열하시오.<br>**

```java
Optional<List<String>> cities = transactions.stream() // Stream<Transaction>
                                            .map(transaction -> transaction.getTrader().getCity()) // Stream<String>
                                            .distinct() // Stream<String>
                                            .collect(toList()); // List<String>

```

3. **케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.<br>**

```java
Optional<List<Trader>> tranderName = transactions.stream() // Stream<Transaction>
                                                .map(Transaction::getTrader) // Stream<Trader>
                                                .filter(trader -> trader.getCity().equals("Cambridge")) // Stream<Trader>
                                                .disticnt() // Stream<Trader>
                                                .sorted(Trander::getName) // Stream<Trader>
                                                .collect(toList()); // List<Trader>

```

4. **모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.<br>**

```java
String NameSortedByAlphabet = transactions.stream()  // Stream<Transaction>
                                    .map(Transaction::getTrader) // Stream<Trader>
                                    .distinct() // Stream<Trader>
                                    .sorted(Trader::getName) // Stream<Trader>
                                    .reduce("",(n1,n2)->n1+n2) // String
```

- 보완점 :<br>
  - 처음부터 거래자의 이름을 가져오기
  - trader로 가져오면 객체자체는 다르기때문에 distinct로 이름이 같아도 중복제거가 안됨.
  - reduce를 사용하면 반복적으로 문자열을 새로만들기때문에 `StringBuilder를 사용하는 joining()`을 사용할 경우 효율성이 좋아짐.

```java
String NameSortedByAlphabet = transactions.stream() // Stream<Transaction>
                                    .map(transaction -> transaction.getTrader().getName()) // Stream<String>
                                    .distinct() // Stream<String>
                                    .sorted() // Stream<String>
                                    .collect(joining()) // String
```

5. **밀라노에 거래자가 있는가?<br>**

```java
boolean isInMilan = transactions.stream() // Stream<Transaction>
                                .map(transaction -> transaction.getTrader()) // Stream<Trader>
                                .anyMatch(trader -> trader.getCity().eqauls("Milan")) // boolean

```

- 보완점<br>
  - map으로 trader를 뽑지않고 anyMatch에서 바로 넘겨줘도 된다.<br>

```java
boolean isInMilan = transactions.stream() // Stream<Transaction>
                                .map() // Stream<Trader>
                                .anyMatch(transaction -> transaction.getTrader().getCity().eqauls("Milan")) // boolean

```

6. **케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오.<br>**

```java
transaction.stream // Stream<Transaction>
            .filter(transaction -> transaction.getTrader().getCity().equals(""Cambridge"")) // Stream<Transaction>
            .map(Transaction::getValue) // Stream<String>
            .foreach(System::println) // 출력
```

7. **전체 트랜잭션 중 최댓값은 얼마인가?<br>**

```java
Optional<Integer> transactionMaxVal = transaction.stream() // Stream<Transaction>
                                    .map(Transaction::getValue) // Stream<Integer>
                                    .reduce(Integer::max) // Integer
```

8. **전체 트랜잭션 중 최솟값은 얼마인가?<br>**

```java
Optional<Integer> transactionMinVal = transaction.stream() // Stream<Transaction>
                                    .map(Transaction::getValue) // Stream<Integer>
                                    .reduce(Integer::min) // Integer

```

- **최솟값 / 최댓값을 구하는 min / max<br>**

```java
// 7.1. Stream을 이용하y는 경우
.max(comparing())

Optional<Integer> transactionMinVal = transaction.stream() // Stream<Transaction>
                                    .max(comparing(Transaction::getValue)) // Integer
// 7.2. IntStream을 이용하는 경우
Optional<Integer> transactionMaxVal = transaction.stream() // Stream<Transaction>
                                    .mapToInt(Transaction::getValue) // IntStream
                                    .max() // Integer
// 8.1. Stream.을 이용하는 경우
.min(comparing())

Optional<Integer> transactionMinVal = transaction.stream() // Stream<Transaction>
                                    .min(comparing(Transaction::getValue)) // Integer
// 8.2. IntStream을 이용하는 경우
Optional<Integer> transactionMinVal = transaction.stream() // Stream<Transaction>
                                    .mapToInt(Transaction::getValue) // IntStream
                                    .min() // Integer
```

<h2> 3. 스트림으로부터 데이터 수집</h2>

- **컬렉션(Collection)**<br>
  : HashMap, HashSet, ArrayList 등과 같은 자료구조
- **컬렉터(Collectors)**<br>
  : 인터페이스로, collect 메서드의 인수
- **컬렉트(collect)**<br>
  : Stream.collect로, 스트림 요소에 리듀싱 연산이 내부적으로 작동됨.

<h3>컬렉터</h3>

1. **카테고리화 한 후 같은 그룹내에서 특정 기준 값 찾기**<br>

- `groupingBy`(분류기준, 분류된 후 같은 그룹에 리듀싱할 작업, 변환함수);<br>
  **_ex. 각 서브 그룹에서 칼로리가 가장 높은 요리 찾기_**

```java
Map<Dish.Type, Dish> theHighestKcal = munu.stream()
                                        .collect(groupingBy(Dish::getType, // 분류 기준
                                                            collectionAndThen(maxBy(comparingInt(Dish::getCalories))), // 리듀싱할 작업(Collector)
                                                            Optional::get  // 변환 타입
                                                            ));

```

2. **두 개의 그룹으로 분할하기**<br>

- `partioningBy`(분류기준, 분류된 후 같은 그룹에 리듀싱할 작업, 변환함수);<br>
  : true, false로 구분하기 때문에 키값은 boolean<br>

  **_ex. 숙제 수행 유무로 그룹 분할화하기_**

```java
Map<Boolean, List<Person>> groupingByHw = people.stream()
                                                .collect(partioningBy(People::isHwDone));

```

<h2> 4. 병렬 데이터 처리와 성능</h2>

<h3>병렬 스트림</h3>
: 하나의 스트림 요소를 각각의 스레드에서 처리하기 위해 여러 단위로 나눈 스트림 .<br>

> 여러 청크로 스트림 나눔 -> 각 스레드에서 분할된 스트림 연산 처리 -> 하나의 스레드에서 연산 결과 합침

- 병렬 스트림을 사용할 때 고려해야할 점<br>

  - **_단순 반복 작업_**은 저수준의 코드가 빠를 수 있음.
  - **_순서에 의존하는 연산_**은 피해야함.<br>
    ex. limit(), findFirst() 등
  - 소량의 데이터일 경우, 레이싱 컨디션이 더 나올 수 있음.
  - 스트림 **_분할이 수월한 자료구조_**인지 판단해야함.
  - **_AutoBoxing과 UnBoxing비용_**도 고려해야함

  <h3>포크/조인 프레임워크</h3>
  : `재귀적으로 작업을 분할하고 결과를 합치는 병렬화 기법`

```java
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
	순차적으로 태스크 계산
} else {
	태스크를 두 서브 태스크로 분할
	태스크가 다시 서브태스크로 분할되도록 이 메서드를 재귀적으로 호출함
	모든 서브태스크의 연산이 완료될 때까지 기다림
	각 서브태스크의 결과를 합침
}
```

- 포크/조인 프레임워크를 사용할 때 고려해야할 점<br>

  - **_join()_** 호출 시 결과가 나올때까지 호출자가 블록됨. 두 서브테스트가 모두 시작된 후 join()호출

  - RecursivaTask에서 ForkJoinPool.invoke()사용X, compute()나 fork() 사용 권장.

  - 왼쪽, 오른쪽 테스크 중 **_한 쪽에는 compute()호출_**하기. 두 서브테스크의 한 테스크에는 같은 스레드를 재사용할 수 있으므로 풀에서 **_태스크 할당 오버헤드를 줄일 수 있음._**

  - **_서브테스크의 실행시간 > 새로운 테스크 포킹 시간_**

  <br>

- **작업훔치기**
  : `작업자 스레드 간에 작업을 균형있게 분배하는 기법.`
  - 각 스레드는 자신에게 할당된 작업을 처리하면서 필요할 때 다른 스레드의 큐에서 작업을 훔침
  - 작업 부하를 균등하게 분산

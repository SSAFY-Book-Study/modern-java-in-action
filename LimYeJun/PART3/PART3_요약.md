## Chapter8 컬렉션 API 개선

### 컬렉션 팩토리

#### 리스트 팩토리
자바에서 적은 요소를 포함하는 리스트를 만드는 방법?
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```

`Arrays.asList()` 팩토리 메서드를 사용하면 코드를 간단하게 줄일 수 있음
```java
List<String> friends
    = Arrays.asList("Raphael", "Olivia", "Thibaut");
```

하지만 위의 방식은 고정 크기의 리스트를 만드는 팩토리 메서드로 `요소를 갱신할 순 있지만 요소를 추가하면 에러 발생`
```java
List<String> friends
    = Arrays.asList("Raphael", "Olivia");
friends.set(0, "Richard");
friends.add("Thibaut");    // UnsupportedOperationException
```
`List.of()` 팩토리 메서드를 이용하면 set 메서드로 아이템을 바꾸려해도 비슷한 예외가 발생한다 (불변성)

#### 집합 팩토리
`List.of()`와 비슷한 방법으로 바꿀 수 없는 집합을 만들 수 있다
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
중복된 요소를 제공해 집합을 만들려고하면 `IllegalArgumentException`이 발생한다
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");  // error
```

#### 맵 팩토리
맵을 만드는 것은 리스트나 집합을 만드는 것에 비해 조금 복잡하다

자바9에서는 두 가지 방법으로 바꿀 수 없는 맵을 초기화할 수 있다
```java
Map<String, Integer> ageOfFriends
  = Map.of("Raphael", 30, "Olivia", 25, "Thibuat", 26);
```
10개 이하의 키와 값 쌍에서는 `Map.of`가 유리
그 이상의 맵에서는 `Map.Entry`객체를 인수로 받는 `Map.ofEntires` 팩토리 메서드를 이용하는 것이 좋다

```java
Map<String, Integer> ageOfFriends
  = Map.ofEntries(entry("Raphael", 30),
                  entry("Olivia", 25),
                  entry( "Thibuat", 26));
```

### 리스트와 집합 처리

- `removeIf`: 프레디케이트를 만족하는 요소를 제거
- `replaceAll`: `UnaryOperator` 함수를 이용해 요소를 바꾼다
- `sort`: 리스트 정렬

#### removeIf 메서드

숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드
```java
for (Transaction transaction : transactions) {
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
        }
    }
```
위 코드는 `ConcurrentModificationException`을 일으킨다
왜? -> 내부적으로 for-each 루프는 Iterator 객체를 사용하므로 위 코드는 다음과 같이 해석 됨

```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
          transactions.remove(transaction);
        }
    }
```
- 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않는다
- Iterator 객체를 명시적으로 사용하고 그 객체의 remove()를 호출함으로 이 문제를 해결할 수 있음

`해결`
```java
for(Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
            iterator.remove();
        }
    }
```
이를 자바8 이후의 코드로 단순하게 바꿀 수 있다

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

`replaceAll`도 마찬가지다 기존의 `stream().map`을 통해 리스트의 각 요소를 새로운 요소로 바꿀 수 있지만 **새 문자열 컬렉션**을 만드는 것이다
Iterator.set 메서드를 이용하면 기존 컬렉션을 바꾸는 것이 가능하며 이를 `replaceAll`로 간단하게 구현할 수 있다

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

### 맵 처리

자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다
자주 사용되는 패턴을 개발자가 직접 구현할 필요가 없도록 이들 메서드를 추가한 것

#### forEach 메서드

`Map.Entry(K, V)`의 반복자를 이용해 맵의 항목 집합을 반복할 수 있다

```java
for(Map.Entry(String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
```
자바 8부터 Map 인터페이스는 BiConsumer(키와 값을 인수로 받음)를 인수로 받는 forEach 메서드를 지원하므로 코드를 간결하게 할 수 있다
```java
ageOfFirends.forEach((friend, age) -> System.out.println(frined + " is " + age + " years old"));
```

#### 정렬 메서드
두 개의 유틸리티를 이용하면 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다

- Entry.comparingByValue
- Entry.comparingByKey

```java
ageOfFirends
      .entrySet()
      .stream()
      .sorted(Entry.comparingByKey())
      .forEachOrdered(System.out::println);
```

#### HashMap 성능
자바 8에서 HashMap의 내부 구조를 바꿔 성능을 개선했다
기존 맵의 항목 키로 해시코드로 접근할 수 있는 버킷에 저장했는데, 최악의 상황에는 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환해야 하므로 성능이 저하된다
따라서 O(log(n))이 걸리는 정렬된 트리구조로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다 
**키가 String, Number 같은 Comparable의 형태여야만 정렬된 트리가 지원됨**

#### getOrDefault 메서드
기존에는 찾으려는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 요청 결과가 널인지 확인해야 한다
- 키가 존재하더라도 값이 널인 상황에선 getOrDefault가 널을 반환할 수 있다

#### 계산 패턴
- computeIfAbsent: 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가
- computeIfPresent: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가
- compute: 제공된 키로 새 값을 계산하고 맵에 저장

키에 해당하는 값이 있거나 없을 때 필요한 계산을 전달할 수 있음

#### 삭제 패턴
제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드는 이미 알고 있다. 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다 
`favouriteMovies.remove(key, value);`

#### 교체 패턴
맵의 항목을 바꾸든 데 사용할 수 있는 두 개의 메서드가 추가되었다
- `replaceAll`: BiFunction을 적용한 결과로 각 항목의 값을 교체한다 -> List의 replaceAll과 비슷한 동작을 수행
- `replace`: 키가 존재하면 맵의 값을 바꾼다 -> 키가 특정 값으로 매핑되었을 때만 교체하는 오버로드 버전도 있다

#### 합침
두 그룹의 연락처를 포함하는 두 개의 맵을 합친다고 가정하면 `putAll`을 사용할 수 있다
```java
Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```
중복된 키가 없다면 잘 동작한다
값을 좀 더 유연하게 합쳐야 한다면 새로운 `merge` 메서드를 이용할 수 있다
두 맵에 `Cristina`가 다른 영화 값으로 존재한다면 `forEach`와 `merge` 메서드를 이용해 충돌을 해결할 수 있다
```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));  // 중복 키가 있다면 연결
```

`merge`를 이용해 초기화 검사를 구현할 수 있다
영화를 몇 회 시청했는지 기록하는 맵이 있을 때, 해당 값을 증가시키기 전에 관련 영화가 이미 맵에 존재하는지 확인이 필요

```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);  // 두 번째 인수 1L은 키와 연관된 기존 값에 합쳐질 널이 아닌 값 또는 값이 없거나 키에 널 값이면 연결
```

### 개선된 ConcurrentHashMap
`ConcurrentHashMap` 클래스는 동시성 친화적이며 최신 기술을 반영한 `HashMap` 버전이다
`ConcurrentHashMap`은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다
따라서 동기화된 `Hashtable` 버전에 비해 읽기 쓰기 연산 성능이 월등

#### 리듀스와 검색
- forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

`ConcurrentHashMap`의 상태를 잠그지 않고 연산을 수행한다는 점에서 이들 연산에 제공한 함수는 **계산이 진행되는 동안 바뀔 수 있는 객체, 값 순서 등에 의존하지 않아야 한다**

### 정리
- 자바 9는 적은 원소를 포함하며 바꿀 수 없는 `리스트`, `집합`, `맵`을 쉽게 만들 수 있도록 컬렉션 팩토리를 지원
- 컬렉션 팩토리가 반화난 객체는 만들어진 다음 바꿀 수 없다
- List 인터페이스는 `removeIf`, `replaceAll`, `sort` 세 가지 디폴트 메서드를 지원
- Set 인터페이스는 `removeIf` 디폴트 메서드를 지원
- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드를 지원
- `ConcurrentHashMap`은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공



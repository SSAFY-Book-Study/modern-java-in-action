# 8장. 컬렉션 API 개선

---

# 컬렉션 팩토리

---

## 팩토리 메서드

---

객체를 생성하기 위해 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지에 대한 결정은 서브클래스가 내리도록 하는 패턴

## 자바의 팩토리 메서드

---

불변 컬렉션을 만드는 메서드

- List.of(Elements)

```java
List<String> fruits = List.of("apple", "banana", "grape");
```

- Map - Map.of(”key”, ”value”,…), Map.ofEntries(entry(”key”, “value”), …)

```java
Map<String> fruits1 = Map.of("apple", "red", "banana", "yellow", "grape", "purple");
Map<String> fruits2 = Map.ofEntries(entry("apple", "red"),
				entry("banana", "yellow"),
				entry("grape", "purple"));
```

### 리스트와 집합 처리

---

**removeIf()**

- 리스트와 집합의 특정 요소를 삭제
- 리스트 요소 삭제 시 ConcurrentModificationException 발생을 막아줌
- 삭제할 요소를 가리키는 프레디케이트를 인수로 받음

**※ConcurrentModificationException**

리스트가 순회하면서 데이터를 삭제하는 도중, 인덱스가 바뀌면서 현재 가리키고 있는 인덱스가 리스트의 크기보다 커지면 발생

**replaceAll()**

- 리스트의 각 요소를 새로운 요소로 바꿈
- 반복자와 set()을 통한 방법보다 안전하며 간단함

### 맵 처리

---

**forEach((K, V) → Void)**

- 맵의 키, 값을 반복함

**Entry.comparingByKey(), Entry.comparingByValue()**

- 맵을 키 또는 값을 기준으로 정렬

**getOrDefault(Key, DefaultValue)**

- 찾으려는 키가 존재하지 않으면 두 번째 인수가 반환됨
- 키가 존재하지 않을 시 null을 반환하는 것을 방지

**computeIfAbsent(), computeIfPresent(), compute()**

- computeIfAbsent : 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가함
- computeIfPresent() : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가함

**remove(key, value)**

- 제공된 키에 해당하는 값 중 특정한 값을 제거함

**replaceAll((K, V) → Void), Replace()**

- List의 replaceAll과 비슷하게 각 항목의 값을 교체
- 키가 존재하면 맵의 값을 바꿈

**merge((K, V) → R)**

- 맵을 합치며 중복된 키가 있을 시, 인수로 받은 BiFunction에 따라 값을 결정

### ConcurrentHashMap

---

- 동시성 친화적이며 여러 기능이 추가된 HashMap

# 9장. 리팩터링, 테스팅, 디버깅

---

## 코드 리팩터링

---

코드 리팩터링을 통해 기존 코드를 간결하고 가독성 높은 코드로 개선

### 익명 클래스 → 람다 표현식

- this의 경우 익명 클래스는 자신을 가리키지만, 람다는 람다를 감싸는 클래스를 가리킴
- 익명 클래스는 내부 변수를 가릴 수 있지만, 람다의 경우 불가능

```java
// 익명 클래스
Runnable r1 = new Runnable(){
	public void run(){
		System.out.println("Hello");
	}
};

// 람다 표현식
Runnable r2 = () -> System.out.println("Hello");
```

### 람다 표현식 → 메서드 참조

- 기존 람다 표현식보다 간결하고 어떤 메서드를 사용하려는지 의도가 명확해짐

```java
// 람다 표현식
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
	menu.stream()
		.collect(
			groupingBy(dish -> {
				if (dish.getCalories() <= 400) return CaloricLevel.DIET;
				else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
				else return CaloricLevel.FAT;
}));

// 메서드 참조
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
	menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```

### 명령형 데이터 처리 → 스트림

- 파이프라인의 의도가 명확해지며, Lazy와 Short Circuit으로 반복자를 사용한 것보다 최적화되어 실행됨

```java
// 반복자를 활용한 명령형 데이터 처리
List<String> dishNames = new ArrayList<>();
	for(Dish dish: menu) {
		if(dish.getCalories() > 300) {
			dishNames.add(dish.getName());
		}
	}

// 스트림으로 구현
menu.parallelStream()
	.filter(d -> d.getCalories() > 300)
	.map(Dish::getName)
	.collect(toList());
```

## 코드 유연성 개선
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

- **List.of(Elements)**

```java
List<String> fruits = List.of("apple", "banana", "grape");
```

- **Map - Map.of(”key”, ”value”,…), Map.ofEntries(entry(”key”, “value”), …)**

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
- 스트림과 비슷한 forEach, reduce, search 연산과 Map을 활용한 연산을 지원
- Map의 일부에만 Lock을 걸어 운용하여 Thread-Safe함

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

---

### 조건부 연기 실행

- 특정 조건에서 작업을 연기하고 실행하여 코드 가독성과 캡슐화 강화

### 실행 어라운드

- 준비, 종료 같은 과정을 반복하는 작업을 람다식으로 변환하여 코드 중복을 줄임

## 람다 테스팅

---

- 코드가 제대로 작동하는지 **단위 테스팅**을 진행해야하지만 람다의 경우 익명 함수이므로 테스트 코드 이름을 호출할 수 없음
- 람다 표현식은 함수형 인터페이스의 인스턴스를 생성하므로 생성된 인스턴스의 동작으로 테스트 가능

**※ 단위 테스트(Unit Test) : 프로그래밍을 할 때 소스코드의 특정 모듈(메서드)이 의도된 대로 정확히 작동하는지 검증하는 절차**

## 디버깅

---

코드를 디버깅 시 **스택 트레이스, 로깅**을 먼저 확인해야하지만 람다의 경우 기존 디버깅 방법이 불가능

### 스택 트레이스

- 람다 표현식의 경우 이름이 없는 익명 함수이기 때문에 복잡한 스택 트레이스가 생성됨
- 현재로서는 이해하기 어려우며, 미래의 자바 컴파일러가 개선해야함

### 로깅

- 스트림의 각 파이프라인을 디버깅하기 위해서는 peek()를 사용

**peek() : 스트림의 파이프라인 사이에 사용하여 스트림의 요소를 확인함(요소를 소비하지 않음)**

# 10장. 람다를 이용한 도메인 전용 언어

---

## DSL(도메인 전용 언어)

---

- 특정 도메인(영역)을 타겟으로 하는 프로그래밍 언어, API
- **코드의 의도**가 명확해야하며, 이해하기 쉽도록 **가독성**이 높아야함
- 자바에서는 Stream, Collectors와 같은 DSL을 제공

### DSL의 장점

- **간결함** : API를 통해 로직의 반복을 줄이고 간결하게 만듦
- **가독성** : 비 도메인 전문가도 이해하기 쉬워 구성원 간 코드와 도메인 영역이 공유됨
- **유지 보수** : 비즈니스 영역에서 요구 사항에 의해 바뀌는 코드를 쉽게 유지 보수가 가능함
- **높은 수준의 추상화** : 도메인과 같은 추상화 수준에서 동작하므로 관련 없는 세부 사항을 숨김
- **집중** : 특정 도메인(비즈니스 등)을 타겟으로 한 언어이므로 특정 코드에 집중하며 생산함
- **관심사 분리** : 애플리케이션 인프라와 관련된 문제와 독립되어 코드에 집중하기 용이

### DSL의 단점

- **DSL 설계의 어려움, 개발 비용** : DSL을 추가하고 유지 보수하는 작업에 많은 비용과 시간이 소모
- **추가 우회 계층** : DSL은 추가적인 계층으로 도메인 모델을 감싸며 이 때 계층을 최대한 작게 만들어 성능 문제를 회피
- **새로 배워야 하는 언어** : 개별 DSL을 사용하는 상황이라면 배워야 할 언어가 하나 추가되는 격
- **호스팅 언어 한계** : 자바와 같이 장황하고 엄격한 문법의 언어는 사용자 친화적 DSL을 만들기 힘듦

  **→ 람다 표현식으로 해당 문제 해결**


### 내부 DSL

- DSL이 사용될 프로그래밍 언어(자바)로 구현한 DSL
- 기존 언어를 이용하기 때문에 새로운 패턴과 기술을 배우지 않고 사용 가능
- 기존 언어로 구현하기 때문에 컴파일러와 외부 DSL을 만드는 도구 없이 함께 컴파일 가능

### 다중 DSL

- **같은 가상 환경(JVM)** 에서 실행되는 언어로 만든 DSL
- 문법적 잡음이 없으며 개발자가 아닌 사람도 코드를 쉽게 이해 가능
- 언어가 혼합되어 있으므로, 빌드 과정 개선 필요와 항상 호환성이 완벽하지 않으므로 호환성 문제 해결 문제

### 외부 DSL

- 자신만의 문법과 구문으로 새 언어(DSL)을 설계
- 새로운 언어을 생성하므로 유연성이 좋지만, 호스트 언어와 분리되어 인공 계층이 생김
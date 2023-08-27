# Part3 - 스트림과 람다를 이용한 효과적 프로그래밍

## Chapter 8 - 컬렉션 API 개선

### 컬렉션 팩토리

배열은 내부적으로 새 요소를 추가하거나 요소를 삭제할 수 없다. 갱신하는 작업을 하는 것은 괜찮지만, 요소를 추가하려고 하면 UnsupportedOperationException이 발생한다.

다음과 같은 방법을 이용할 수 있다.

```java
Set<String> friends
    = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibuat"));

Set<String> friends
    = Stream.of("Raphael", "Olivia", "Thibuat")
        .collect(Collectors.toSet());
```

하지만, 두 가지 방법 모두 매끄럽지 못하며 내부적으로 불필요한 객체 할당을 필요로 한다.

> 자바에서는 큰 언어와 관련된 비용이 든다는 이유로 리터럴 컬렉션을 지원하지 못했다. 자바 9에서는 대신 컬렉션 API를 개선했다.

### 리스트 팩토리

List.of 팩토리 메서드를 이용해 간단히 리스트를 만들 수 있다.

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibuat");
```

위 코드에 요소를 삽입하려고 하면 위에서 살펴본 예외가 발생한다. 사실 **변경할 수 없는 리스트**가 만들어졌기 때문이다.

> 어떤 상황에서 새로운 컬렉션 팩토리 메서드 대신 스트림 API를 사용해 리스트를 만들어야 하는지 궁금할 것이다.
Collectors.toList() 컬렉터로 스트림을 리스트로 변환할 수 있다. 데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 없다면 사용하기 간편한 팩토리 메서드를 이용할 것을 권장한다.

### 리스트와 집합 처리

자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드를 추가했다.

- removeIf
- replaceAll
- sort

이들 메서드는 **호출한 컬렉션 자체를 변경**한다.

#### removeIf method

```java
// 솟주로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드
transactions.removeIf(transaction -> 
        Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

#### replaceAll method

```java
// 리스트의 각 요소를 새로운 요소로 바꾸는 코드
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)+code.subString(1)));
```

#### 맵 처리

- forEach method

```java
// 반복하는 방법
ageOfFriends.forEach((friend, age)->System.out.println(friend + "is" + age + "years old."));
```

### getOrDefault method

기존에는 찾으려고 하는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 요청 결과가 널인지 확인해야 한다. 기본값을 반환하는 방식으로 이 문제를 해결할 수 있다.
이 메서드는 첫 번째 인수로 키를, 두 번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다.

```java
Map<String, String> favouriteMovies
    = Map.ofEntries(entry("Raphael", "Star Wars"),
        entry("Olivia", "James Bond"));

System.out.println(favouritesMovies.getOfDefault("Olivia", "Matrix")); // print James bond
System.out.println(favouritesMovies.getOfDefault("Thibuat", "Matrix")); // print matrix
```

#### 계산 패턴

> 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요할 때가 있다. 예를 들어 키를 이용해 값비싼 동작을 실행해서 얻은 결과를 캐시하려고 하면, 키가 존재할 때 결과를 다시 계산할 필요가 없다.

- computeIfAbsent
- computeIfPresent
- compute

### 개선된 ConcurrentHashMap

> ConcurrentHashMap 클래스는 동시ㅈ성 친화적이며 최신 기술을 반영한 HashMap이다. ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 도잇 추가, 갱신 작업을 허용한다. 따라서 동기화된 Hashtable 번에 비해 읽기 쓰기 연산 성능이 월등하다.

#### 리듀스와 검색

ConcurrentHashMap은 스트림에서 봤던 것과 비슷한 종류의 세 가지 새로운 연산을 제공한다.

- forEach
- reduce
- search

이들 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다는 것을 기억하자.

#### 계수

ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다.
기존의 size 메서드 대신 새 코드에서는 long을 반환하는 mappingCount 메서드를 사용하는 것이 좋다. 그래야 매핑의 개수가 int의 범위를 넘어서는 이후의 상황을 대처할 수 있기 때문이다.

## Chapter 9 - 리팩터링, 테스팅, 디버깅

### 가독성과 유연성을 개선하는 리팩터링

> 코드 가독성이란 무엇을 의미할까? 코드 가독성이 좋다는 것은 '어떤 코드를 다른 사람도 쉽게 이해할 수 있다'는 것을 의미한다. 자바 8의 새 기능을 이용해 코드의 가독성을 높일 수 있다.

- 익명 클래스를 람다 표현식으로 리팩터링하기
- 람다 표현식을 메서드 참조로 리팩터링하기
- 명령형 데이터 처리를 스트림으로 리팩터링하기

#### 익명 클래스를 람다 표현식으로 리팩터링하기

```java
Runnable r1 = new Runnable(){
    public void run(){
        System.out.println("Hello");
        }
}

// -> to lambda

Runnable r2 = () -> System.out.prinln("Hello");
```

하지만 모든 익명 클래스를 람다로 변환할 수 있는 것은 아니다. 첫째, 익명 클래스에서 사용한 this, super는 람다식에서 다른 의미를 가진다.

익명 클래스에서 this는 익명 클래스 자신을 가리키지만, 람다에서 this는 람다를 감싸는 클래스를 가리킨다.

둘째, 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다. 하지만 람다식에서는 가릴 수 없다.

마지막으로 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.

#### 람다 표현식을 메서드 참조로 리팩터링하기

> 람다 표현식은 쉽게 전달할 수 있는 짧은 코드다. 하지만 람다표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.

#### 명령형 데이터 처리를 스트림으로 리팩터링하기

> 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다. 스트림은 앞 장에서 설명한 것처럼 쇼트서킷과 게으름이라는 강력한 최적화뿐 아니라 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공한다.

예를 들어 다음 코드는 두 가지 패턴(필터링과 추출)로 엉킨 코드다. 이 코드를 접한 프로그래머는 전체 구현을 자세히 살펴본 이후에야 전체 코드의 의도를 이해라 수 있다. 게다가 이 코드를 병렬로 실행시키는 것은 매우 어렵다.
```java
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu){
    if(dish,getCalories() > 300){
            dishNames.add(dish.getName());
        }
}

// -> using stream api

menu.parallelStream()
        .filter(d -> d.getCalories() > 300)
        .map(Dish::getNmae)
        .collect(toList());
```

위와 같이 스트림을 이용하면 문제를 더 직접적으로 기술할 수 있을 뿐 아니라 쉽게 병렬화가 가능하다.

### 람다로 객체지향 디자인 패턴 리팩터링하기

> 다양한 패턴을 유형별로 정리한 것이 디자인 패턴이다. 디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 **재사용**할 수 있는, 검증된 청사진을 제공한다.

디자인 패턴에 람다 표현식이 더해지면 색다른 기능을 발휘할 수 있다.

#### 전략

> 전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.

#### 템플릿 메서드

> 템플릿 메서드는 '이 알고리즘을 사용하고 싶은데 그대로는 안되고 조금 고쳐야 하는' 상황에 적합하다.

#### 옵저버

> 어떤 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.

#### 의무 체인

> 작업 처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다. 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식이다.

#### 팩토리

> 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 떄 사용하는 패턴이다.

### 람다 테스팅

일반적으로 좋은 소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 **단위 테스팅**을 진행한다.

```java
public class Point {
    private final int x;
    private final int y;
    
    private Point(int x, int y){
        this.x = x;
        this.y = y;
    }
    
    public int getX() return x;
    public int getY() return y;
    
    public Point moveRightBy(int x){
        return new Point(this.x + x, this.y);
    }
}

// unit test

@Test
public void testMoveRightBy() throws Exception {
    Point p1 = new Point(5, 5);
    Point p2 = p1.moveRightBy(10);
    
    assertEquals(15, p2.getX());
    assertEquals(5, p2.getY());
}
```

### 디버깅

문제가 발생한 코드를 디버깅할 때 개발자는 스택 트레이스와 로깅을 확인해야 한다.

#### 스택 트레이스 확인

람다 표현식은 이름이 없기 때문에 복잡한 스택 트레이스가 생성된다. 메서드 참조를 사용해도 스택 트레이스에는 메서드명이 나타나지 않는다.

#### 정보 로깅

forEach 등을 이용해 스트림 결과를 출력하거나 로깅할 수 있다.

## Chapter 10 - 람다를 이용한 도메인 전용 언어

> 도메인 전용 언어(DSL)로 애플리케이션의 비즈니스 로직을 표현함으로써 버그와 오해를 미리 방지할 수 있다.

### 도메인 전용 언어

DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어다.DSL의 경우 동작과 용어는 특정 도메인에 국한되므로 다른 문제는 걱정할 필요가 없고 오직 자신의 앞에 놓인 문제를 어떻게 해결할지에만 집중할 수 있다.

- 우리의 코드의 의도가 명확히 전달되어야 한다.
- 한 번 코드를 구현하지만 여러 번 읽는 것을 방지한다.

#### DSL pros and cons

- pros : 간결함, 가독성, 유지보수, 높은 수준의 추상화, 집중, 관심사 분리
- cons : DSL 설계의 어려움, 개발 비용, 추가 우회 계층, 새로 배워야 하는 언어, 호스팅 언어의 한계

### 최신 자바 API의 작은 DSL

> 자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다. 자바 8 이전의 네이티브 자바 API는 이미 한 개의 추상 메서드를 가진 인터페이스를 가지고 있었다. 하지만 무명 내부 클래스를 구현하려면 불필요한 코드의 작성이 불가피했다.

```java
// 나이와 이름을 비교하는 코드
person.sort(comparing(Person::getAge)
        .thenComparing(Person::getName));
```

#### 메서드 체인
> 이 방법을 이용하면 한 개의 메서드 체인 호출로 데이터를 정의할 수 있다.

빌더를 구현해야 한다는 것이 메서드 체인의 단점이다. 상위 수준의 빌더를 하위 수준의 빌더와 연결할 접착 많은 접착 코드가 필요하다. 도메인의 객체 중첩 구조와 일치하게 들여쓰기를 강제할 방법이 없다는 것도 단점이다.

#### 중첩된 함수 이용

> 이 방법은 다른 함수 안에 함수를 이용해 도메인 모델을 만든다. 

```java
Order order = order("BigBank", 
        buy(80,
        stock("IBM", on("NYSE")), at(125.00)),
        sell(50,
        stock("GOOGLE", on("NASDAQ"), at(375.00))
        ));
```

이 방법에서의 문제점은 결과 DSL에 더 많은 괄호를 사용해야 한다는 사실이다. 더욱이 인수 목ㄹ록을 정적 메서드에 넘겨줘야 한다는 제약도 있다.

#### 람다 표현식을 이용한 함수 시퀀싱

> 이 방법은 람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현해야 한다.

이 패턴은 이전 두 가지 DSL 형식의 두 가지 장점을 더한다. 메서드 체인 패턴처럼 플루언트 방식으로 거래 주문을 정의할 수 있다. 또한 중첩 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다.

안타깝게도 많은 설정 코드가 필요하며 DSL 자체가 자바 8 람다 표현식 문법에 의한 잡음의 영향을 받는다는 것이 이 패턴의 단점이다.

> DSL의 주요 기능은 개발자와 도메인 전문가 사이의 간격을 좁히는 것이다.

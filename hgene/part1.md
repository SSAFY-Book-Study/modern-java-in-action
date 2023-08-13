# 1장

## 자바의 발전 방향

1. 간결한 코드
2. 병렬 실행환경의 쉬운 활용

---

## 자바8의 변화 내용

### [ 람다, 메서드 참조 ]

> 메서드와 함수를 값으로 취급하도록 하는 것
>

- 메서드를 일급값으로 취급하는 것 : **[메서드 참조](##-람다-표현식-상세)** ( ↔ 객체참조 )
    - 문제 자체를 더 직관적으로 설명 가능해짐
    - 어떤 메서드의 동작이 복잡하다면( 코드길이가 길다면 ) 메서드를 정의하고, 메서드 참조를 활용하는 것이 바람직
- 함수를 일급값으로 취급하는 것 : **[람다 표현식](##-메서드-참조-상세)**
    - 이용할 수 있는 편리한 클래스나 메서드가 없다면, 훨씬 간결하게 사용가능
        - 한번만 사용할 메서드는 굳이 정의할 필요가 없지 않은가!

---

### [ 동작 파라미터화 ]

> 메서드를 다른 메서드의 매개변수로 제공하는 것 → 스트림 API의 기초 원리
>

- 동작 파라미터화
    - 자주 변경되는 요구사항에 효과적으로 대응 가능 → 유연성 & 재사용성 확보
    - 원하는 동작을 메서드의 파라미터로 전달하기 때문에 이를 전달받는 메서드는 런타임시에 실행을 확정
    - 동작 파라미터화 구현시, 코드가 지저분해질 수 있음 → 람다 표현식으로 간결하게 만들어 줄 수 있음
    - 동작 파라미터화를 과정

      (1) 거의 비슷한 코드가 반복된다면, 그 코드를 추상화한다

      (2) 반복되는 메서드 간의 차이가 값이라면, 우선 값을 파라미터화 시키고 동작이라면 메서드를 파라미터화 시켜야 한다

      (3) 그 결과 생성된 코드는 객체를 통해 내부 파라미터( 메서드 )에 접근해야하므로 지저분할 수 있다. 따라서 람다 표현식 등을 통해 코드를 간결하게 만들어 준다.


---

### [ 스트림 API ]

> 작업을 고수준으로 추상화한 일련의 스트림으로 만들어 한번에 처리할 수 있도록 함 → 속도 & 안정성 확보
>

- 물리적인 순서로 한번에 하나씩 운반하지만, 각각의 작업장에서는 동시에 작업처리
- 컬렉션과 다른 방식으로 데이터 처리


    스트림 API
    
    - 내부 반복 : 라이브러리 내에서 반복작업 처리
        - 자주 반복되는 패턴( 필터링, 추출, 그룹화 ) 제공
    - 멀티 코어를 통한 작업 병렬처리
        - 일부 CPU는 컬렉션 앞부분 작업 처리, 일부는 뒷부분 처리( 포킹단계 ) 후 하나로 합침
        - 안정적인 가변 자원 공유 환경 제공
    
    컬렉션
    
    - 외부 반복 : 반복 작업 직접 처리
    - 큰 단위의 컬렉션 작업도 단일 CPU가 수행 → 느림

---

### [ 디폴트 메서드 ]

> 구현 클래스에 구현하지 않아도 되는 메서드를 인터페이스에 추가하는 기능
>

- 인터페이스의 변경을 용이하게 해줌 ↔ 기존에는 해당 인터페이스의 구현코드를 모두 수정해야 했음
- 기존 구현 코드를 수정하지 않아도, 이미 공개된 인터페이스의 수정이 가능해짐

---

### [ Optional ]

> NullPointer 에 대한 예방 책
>

---

## 람다 표현식 상세

- 특징
    - 익명 메서드를 간결한 버전
    - 특정 클래스에 종속되지 않음 → 함수라고 표현
    - 메서드의 인수, 변수로 저장 가능
- 사용
    - 함수형 인터페이스의 인스턴스를 취급
        - 함수형 인터페이스 : 오직 1개의 추상 메서드 존재( 디폴트 메서드는 상관 X )
        - @FunctionalInterface : 함수형 인터페이스 애노테이션( 컴파일러 체크 )
    - 예외를 던지는 람다 생성 방법
        - 확인된 예외를 선언하는 함수형 인터페이스 직접 정의
        - 람다를 try~catch 블록으로 잡아야 함
    - 비효율적인 오토박싱 대체 방법
        - 오토박싱 : 제네릭 사용시, 기본형 ↔ 참조형 간의 박싱, 언박싱 과정이 자동으로 이루어지는 것
        - 함수형 인터페이스를 제공해서 오토박싱을 피할 수 있도록 함
    - 유효한 람다 표현식

        ```java
        /** 
         * (파라미터) -> { 바디; }
         * (파라미터) -> 바디 : 실행문 하나는 중괄호 생략가능
         * return이 함축되어 있으므로, 명시적으로 사용할 필요 
         */
        (String s) -> s.length
        (Apple a) -> a.getWeight() > 150
        (int x, int y) -> {
        	System.out.println("Result:");
        	System.out.println(x + y);
        }
        () -> 42
        (Apple a1, Apple a2) -> a1.getWeigth().compareTo(a2.getWeight);
        ```

        ```java
        /** 
         * 함수형 인터페이스
         */
        Predicate<T> : T → boolean
        
        Comparator<T> : (T, T) → int
        
        Runnable :   () → void
        
        ActionListner : () -> void
        
        Callable<V>  :  () → V
        
        PrivilegedAction<T> : () -> T
        
        Consumer<T> : T → void 
        
        Supplier<T> :  () → T
        
        BiPredicate<T,U> : (T,U) → boolean
        
        IntBinaryOperator : (int, int) → int
        
        Function<T, R> : (T) -> R
        ```

        ```java
        /** 
         * 예외를 던지는 람다 생성법 : try~with~resources 예제
         */
        @FunctionalInterface
        public interface BufferedReaderProcessor {
        	String process(BufferedReader br) throws IOException;
        }
        BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
        
        Function<BufferedReader, String> f = (BufferedReader br) -> {
        	try {
        		br.readLine();
        	} catch(IOException e) {
        		throw new RuntimeException(e);
        	}
        }
        ```

        ```java
        /** 
         * 오토박싱 대체
         * IntPredicate, IntConsumer, LongBinaryOperator, IntFunction, ...
         */
        public interface IntPredicate {
        	boolean test(int t);
        }
        
        IntPredicate evenNumbers = (int i) -> i % 2 == 0;
        ```


- 컴파일러가 람다형식( 대상형식 )을 확인하는 과정
    - 형식 검사
        1. 람다의 함수형 인터페이스 확인
        2. 제네릭은 특정 타입으로 대치
        3. 함수 디스크립터 일치여부 확인
    - 형식 추론
        - 람다 표현식이 사용된 컨텍스트를 통해 파라미터 형식을 추론할 수 있음 → 파라미터 타입 생략가능
            - 상황에 따라 장단점이 다름
    - 지역변수 배제
        - 람다 표현식에서는 한번만 할당되는( final 혹은 final처럼 사용되는 ) 지역변수만 참조 가능
            - 이미 할당 해제된 변수를 참조하는 것을 방지하기 위해 람다표현식에 지역변수의 복사본을 제공하기 때문( 변수 캡쳐 )
- 클로저(Clojure)
    - 외부 함수안에 있는 내부 함수가 외부함수의 지역변수를 사용할 수 있음
    - 공통점( 람다 ) : 외부 함수가 종료되더라도 내부함수에서 참조하는 외부함수의 context는 유지
        - 클로저가 생성되는 시점에 함수 자체가 복사되어 따로 컨텍스트를 유지하기 때문에 (1) 에서 이미 외부 함수가 종료되었음에도 (2)에서 외부함수의 지역변수 참조 가능 → 컴파일러가 변수 캡쳐를 해주기 때문에, 변수는 final 혹은 유사 final이어야 함

    ```java
    public class closure {
        @Test
        void closure() {
            final var supplier = outerMethod(); //(1)
            System.out.println(supplier.get()); //(2)
        }
     
        private Supplier<String> outerMethod() {
            final String str = "outer method local variable";
            return () -> str;
        }
    }
    ```

    - 차이점
        - 클로저는 외부 함수의 지역변수에 접근해서 값을 변경할 수 있음
            - 클로저는 외부 변수를 참조 ↔ 람다는 자신이 받는 매개변수(외부변수)만을 참조

---

## 메서드 참조 상세

- 특징
    - 람다식의 간결한 버전
    - 기존의 메서드 정의를 재활용해서 람다처럼 사용 → 가독성 확보
- 사용
    - 클래스, 인스턴스, 정적 클래스 단위로 메서드 참조 가능
    - new 키워드를 통해 객체 생성 가능
    - this 키워드를 통해 참조 가능

    ```java
    /** 
     * 정적 클래스, 클래스, 인스턴스 메서드 참조
     */
    Classname::staticMethod;
    
    Classname::method;
    
    instanceVar::method;
    
    this::method;
    
    Classname::new;
    ```

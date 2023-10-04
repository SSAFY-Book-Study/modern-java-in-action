# Part4

## Ch.2 null 대신 Optional 클래스

- 기존에는 값이 없는 상황이 존재한다면 매번 null인지 확인
    
    ```java
    public String getCarInsuranceName(Person person) {
    	if (person != null) {
    		Car car = person.getCar();
    		if (car != null) {
    			Insurance insurance = car.getInsurance();
    			if (insurance != null) {
    				return insurance.genName();
    			}
    		}
    	}
    	return "unknown";
    }
    ```
    
- null의 문제
    - 에러의 근원
    - 코드 가독성 저하
    - null 자체의 의미 x
    - 자바 철학에 위배 : 자바는 개발자로부터 모든 포인터를 숨겼으나 null 포인터는 예외
    - 형식 시스템에 구멍을 생성 : null은 무형식이며 정보를 포함하지 않으므로 모든 형식에 null을 할당 가능 → 시스템의 다른 부분으로 null이 퍼졌을 때 null이 어떤의미로 사용되었는지 알 수 x
- 옵셔널 클래스
    - Optional은 선택형값을 캡슐화하는 클래스
        - 값이 있으면 값을 감싸고, 값이 없으면 Optional.empty 메서드로 Optional을 반환
- Optional 적용 패턴
    - 옵셔널 객체 만들기
        - 빈 옵셔널
            - Optional.empty()
        - null이 아닌 값으로 Optional 만들기
            - Optional.of()
        - null값으로 Optional 만들기
            - Optional.ofNullable()
    - 맵으로 Optional값 추출
        - map() 사용
    - flatMap으로 Optional 객체 연결
        - 중첩 Optional 구조일 때는 flatMap() 사용하여 일차원으로 평준화
- Optional 스트림 조작
    - 자바9에서 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드 추가
- 디폴트 액션과 Optional 언랩
    - get()
        - 값이 없을 때, NoSuchElementException을 발생, 값이 확실한 거 아니면 권장x
    - orElse(T other)
        - 기본값 제공
    - orElseGet(Supplier<? extends T> other)
        - 값이 없을 때, Supplier 실행
- 필터로 특정값 거르기
    - 프레디케이터를 인수로 받음

| 메서드 | 설명 |
| --- | --- |
| empty | 빈 Optional 인스턴스 반환 |
| filter | 값O → 프레디케이트 일치 → 값을 포함하는 Optional 반환
값X || 프레디케이트 불일치 → 빈 Optional 반환 |
| flatMap | 값O → 인수로 제공된 함수를 적용한 결과 Optional 반환
값X → 빈 Optional 반환 |
| get | 값O → Optional이 감싸고 있는 값 반환
값X → NoSuchElementException 반환 |
| ifPresent | 값O → 지정된 Consumer 실행
값X → 아무일 X |
| ifPresentOrElse | 값O → 지정된 Consumer 실행
값X → 아무일 X |
| isPresent | 값O → true
값X → false |
| map | 값O → 제공된 매핑 함수 |
| of | 값O → 값을 감싸는 Optional 반환
값X → NullPointerException |
| ofNullable | 값O → 값을 감싸는 Optional 반환
값X → 빈 Optional 반환 |
| or | 값O → 같은 Optional 반환
값X → Supplier에서 만든 Optional 반환 |
| orElse | 값O → 값
값X → 기본값 |
| orElseGet | 값O → 값
값X → Supplier에서 제공하는 값 반환 |
| orElseThrow | 값O → 값
값X → Supplier에서 생성한 예외 발생 |
| stream | 값O → 값만 포함하는 스트림 반환
값X → 빈 스트림 반환 |

## Ch.2 새로운 날짜와 시간 API

- LocalDate, LocalTime, Instant, Duration, Period 클래스
    - LocalDate, LocatTime
        - 정적 팩토리 메서드 of로 인스턴스 생성
        - 정적 메서드 pares로도 생성
            
            ```java
            LocalDate date = LocalDate.parse("2017-09-21");
            ```
            
        - 파싱 불가능 → DateTimeParseException
    - 날짜와 시간 조합
        - LocalDateTime
    - Instance 클래스 : 기계의 날짜와 시간
        - 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현
        - 팩토리 메서드 ofEpochSecond
    - Duration과 Period 정의
        - 앞에 모든 클래스는 Temporal 인터페이스 구현
        - Duration
            - 정적 팩토리 메서드 between으로 두 LocalTime, LocalDateTime, Instant 사이의 지속시간 알 수 있음
            - 시간 단위 표현으로 LocalDate 불가
        - Period
            - 년, 월, 일로 시간을 표현할 때 사용
    - 위의 클래스들은 모두 불변
- 날짜 조정, 파싱, 포매팅
    - with, get, plus, minus 메서드로 Temporal을 특정 시간만큼 앞뒤로 이동시킬 수 있음
    - 복사 방식이라 할당하지 않으면 아무 일도 x
    - TemporalAdusters 사용하기
    - 날짜와 시간 객체 출력과 파싱
        - DateTimeFormatter 사용
- 다양한 시간대와 캘린더 활용 방법
    - 시간대
        - ZoneRules 클래스에는 약 40개 정도의 시간대
        - 지역 ID로 ZoneID 구분
    - UTC/Greenwich 기준의 고정 오프셋
        - ZoneOffset으로 자오선과 런던 그리니치와 시간값의 차이 표현 가능 → 서머타임 미적용으로 추천x

## Ch.13 디폴트 메서드

- 반환 형식 앞에 default를 붙여 새로 생긴 메서드의 추가 구현 없이도 인터페이스를 구현할 수 있도록 도움
- 디폴트 메서드?
    - default 키워드로 시작, 메서드 바디를 포함
    - 인터페이스 상속받는 모든 클래스는 디폴트 메서드 상속 → 소스 호환성 유지
- 디폴트 메서드 활용 패턴
    - 선택형 메서드
        - 인터페이스에 디폴트 메서드를 정의하여 불필요한 빈 구현을 방지
    - 동작 다중 상속
        - 인터페이스를 통하여 다중 상속 구현
- 해석 규칙
    - 같은 시그니처를 갖는 디폴트 메서드를 상속 받을 시
        1. 클래스가 항상 이긴다.
        2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. B가 A를 상속받으면 B가 이긴다.
        3. 여전히 우선순위가 결정되지 않았다면, 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.

## CH.14 자바 모듈 시스템

- 압력 : 소프트웨어 유추
    - 관심사 분리
        - 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙
        - SoC 장점
            - 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업 가능
            - 개별 부분을 재사용 용이
            - 전체 시스템 쉽게 유지보수
    - 정보 은닉
        - 세부 구현을 숨기도록 장려하는 원칙
- 자바 모듈 시스템을 설계한 이유
    - 모듈화의 한계
        - 제한된 가시성 제어
        - 클래스 경로
            - 약점
                - 클래스 경로에는 같은 클래스를 구분하는 버전 개념x
                - 클래스 경로는 명시적인 의존성 지원x
    - 거대한 JDK
        - 높은 유지 비용과 진화를 방해하는 문제 존재
- [module-info.java](http://module-info.java) 파일은 모듈의 이름을 저장하며 필요한 의존성과 공개 API를 정의
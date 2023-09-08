# 4week issue에 대한 토론 일지

---
## 회의 시나리오
복붙용

## 이슈

1. 안건을 올린 이유 발표
  
2. 안건에 대해 답변자 발표 및 설명 요약
  
3. Q n A 및 회의록 기록

---
## 이슈

1. 안건을 올린 이유 발표
- Java의 null 처리 방법에는 Optional 이외에도 requiredNonNull, assert 등이 있는 것으로 알고 있습니다.
이들과 Optional은 각각 어떻게 사용되는지 차이점이나 특징, 사용사례 등에 대해 알고싶습니다.
  
2. 안건에 대해 답변자 발표 및 설명 요약
- 자바에서 null은 아주 골칫덩어리 : 자바 애플리케이션에서 가장 많이 발생하는 예외가 NPE 발생.
- null을 피하려면?
  - 매번 분기 ? : 잊을 수도 있고, 너무 코드가 복잡해짐 => null handling 필요함.
- null handling 방법 :
  - 선언과 동시에 초기화.
  - Objects.isNull(), Objects.nonNull() 사용 : if(var == null) 사용 지양 권장.
  - Objects.requireNonNull() : 가독성을 위함.
  - 타입으로 관리하기 : 레퍼런스 타입만 null을 가지기 때문에 되도록 primitive 타입 사용 권장.
  - 메서드 오버로딩 : 오버로딩을 활용하여 필요한 인자만 들어가는 메서드를 정의할 것.
  - Collections.emptyList() : Collection을 반환하는 메서드에서는 Collections.emptyList() 반환.
  - Optional :
    - null을 요소로 넣지 말 것 -> empty 활용.
    - ofNullable()을 사용하여 null 처리할 것.
- OPtional 사용시 주의사항 :
  - 멤버 변수에 Optional 선언은 안티 패턴 : 직렬화 불가 -> 메서드 반환 목적으로만 사용할 것.
  - collection에는 optional을 사용하지 말것 : Collections.empty..() 활용할 것.
  
5. Q n A 및 회의록 기록


---
## 이슈

1. 안건을 올린 이유 발표
  
2. 안건에 대해 답변자 발표 및 설명 요약
  
3. Q n A 및 회의록 기록


---

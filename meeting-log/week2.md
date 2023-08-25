# 1week issue에 대한 토론 일지
### 회의 시나리오
1. 안건을 올린 이유 발표

2. 안건에 대해 답변자 발표 및 설명 요약

3. Q n A 및 회의록 기록

- 회의록은 질문한 사람이 직접 글로 남겨서 기록합니다.

---
## 이슈 2 [맵리듀스 feat. 대용량 데이터 처리 방식]
1. 안건을 올린 이유 발표
- 대용량 데이터 처리에 관한 논의는 백엔드 엔지니어에게 중요한 이슈이기 때문에 대용량 데이터 처리에까지 한 번 정리하는 시간을 가져보면 좋을 것 같았습니다.

2. 안건에 대해 답변자 발표 및 설명 요약
- 맵리듀스 패턴이란 :
  - 대용량 데이터를 어떻게 효율적으로 처리하는가? 분할해서 병렬로 처리하면 효율적 -> 이 생각을 기반으로 나온 개념
  - 데이터를 매핑하는 느낌으로 우선 분할하고 -> 리듀싱을 하여 분할된 데이터를 다시 집계
 
- 원리( 하둡 ) :
  - 저장해둔 데이터 입력받기
  - 데이터 매핑( 분할 )
  - 같은 키를 기준으로 데이터 집계( 셔플링 & 소팅 )
  - 분할된 데이터의 집계 결과를 다시 병합( 리듀싱 )
 
- 왜 대용량 데이터 처리에 적합한가?
  - 읽어서 집계만 하기 때문에 -> 상태변화 x -> 병렬로 처리 가능
  - 클러스터링도 가능
 
- 데이터 특성에 따른 처리
  - 맵리듀스 방식은 이미 저장된 결과만 활용 -> 배치 처리( 정적인 데이터 )에 적합한 방식
  - 데이터베이스 : B Tree 자료구조 -> 범위탐색에 유리
  - 실시간 스트리밍 : 스트림 자료구조( Ex. Apache Kafka ), Tree 자료구조 등

3. Q n A 및 회의록 기록
- Q. 맵리듀스의 "맵"이란 어떤 의미를 가지는가?
- A. 키-값의 구조로 매핑하는 Java의 HashMap과 유사한 의미를 가진다.
- Q. B Tree가 무엇인가?
- A. 일반 이진트리의 사향트리가 될 위험성을 제거하고자 균형잡힌 형식으로 트리 자료구조 형성 -> 이에 따라 O(log N)의 시간복잡도를 가진다.
---
## 이슈3 [Collectors.method() vs Stream.method()]
1. 안건을 올린 이유 발표

2. 안건에 대해 답변자 발표 및 설명 요약
- reduce()와 Collectors.reducing()은 순차처리의 경우 실행시간차이는 없지만 가독성과 복잡한 데이터 처리나 데이터 분석 추가 처리를 할경우엔 Collectors.reducing()을 사용이 좋다.
- reduce()
    : 시간 과정이 지나면서 데이터를 추출, 스트림의 요소들을 수집(collect)하여 결과를 생성하는 데에 사용
- collectors.reducing
  : 가독성이나 복잡한 데이터를 처리할때 좋음
- count()
  : 결과 추출, 반환값은 long
  
병렬처리 과정에서 순서차이가 있음, 병렬처리의 경우 - 연산계산 시 스트림마다 결과가 달라질 수 있기 때문에 유의해야함

   . Q n A 및 회의록 기록
Q. 예시 코드에서 map 중간연산을 스킵된 이유
A.데이터 원본자체에서 사이즈의 변경이 없기 때문에 연산최적화로 인해 스킵된것이다.
- 회의록은 질문한 사람이 직접 글로 남겨서 기록합니다.

---
## 이슈3 [Collectors.method() vs Stream.method()]
1. 안건을 올린 이유 발표

2. 안건에 대해 답변자 발표 및 설명 요약
- 안쓰는 이유는 코딩테스트에서는 시간 복잡도가 중요하다. 직접 시간을 측정하였지만, 외부 반복과 stream은 비슷하다. 하지만 parallelstream의 경우 전자의 경우보다 시간이 더 오래걸렸다.
- 특정 상황(많은 데이터를 그룹핑하는)에서는 parallelStream이 효율적이다.

3. Q n A 및 회의록 기록
- stream으로 변환하는 비용 때문에 오버헤드가 발생하고, 복잡한 연산이 아닌 경우 단순 순회가 더 효율적이다.
- parallel의 경우 컴퓨터의 상황을 알고 사용해야하는데, 코딩테스트의 경우 서버의 하드웨어 상황을 모르기 때문에 병렬 처리 시 효과를 보장할 수 없으므로 단순 순회가 더 효율적이다.
---

## 이슈5 [Optional을 사용하는 이유]
1. 안건을 올린 이유 발표
- 개발에서 사용하는 Optional에 대해 한번더 짚어보자.
  
2. 안건에 대해 답변자 발표 및 설명 요약
- 옵셔널 널 반환을 막기 위해 제작.
- 코드의 지저분함을 제거하고 받아온 값을 바로 처리가 가능
- null 할당 금지
- 기본값 반환
- Optinal 직렬화 안됨 -> 함수의 반환값으로만 되도록이면 사용
  
3.  Q n A 및 회의록 기록
- 안전하다. NullPointerException 피할 수 있다.
- 빈 껍대기 안에 뭐가 있는가?
- Optional.of / Optional.ofNullable() 로 생성.

---

## 이슈6 [Collectors.toMap에 대해 궁금합니다]
1. 안건을 올린 이유 발표
- 그룹화 메서드에서 groupingBy, partitioningBy를 통해 그룹화를 하는데, 원래 Collectors.toMap으로 그룹화하는 방식을 알고있었지만 책에서 다루지 않아 어떤 차이가 있는지 궁금했습니다. 
  
2. 안건에 대해 답변자 발표 및 설명 요약

Collectors에서 제공하는 메서드는 컬렉션을 반환하는 메서드들이 있ek
toMap은 Map을 반환하지만 4가지 인자를 이해하고 사용해야 한다
- 인자 : keyMapper 와 valueMapper 정의 필요
- toMap 메서드는 중복을 자동으로 필터링하지 X → 키값이 중복될 경우 IllegalStateException
- mergeFunction() : 동일한 키로 인해 충돌 발생시, 어떠한 value를 취할 것인지 결정할 때 사용
- Supplier ****mapFactory : Collectors.toMap() 메서드를 통해 LinkedHashMap( 순서유지 )을 생성하고 싶은 경우 4st 인자로 사용 가능
toMap을 하면 중복키에 대한 추가적인 처리가 필요할 수 있다

단점
- Key값 중복 시 에러
  - 해결 방법 MergeFunction
- 새로운 key값에 대한 ArrayList 오버헤드
- 병합 연산에 의한 성능 저하 

groupingBy() & partitioningBy()
partitioningBy: true, false 키 값이 정해진 특수한 그룹화로 groupingBy와 유사

groupingBy는 그룹화를 목적으로 만들었기 때문에 그룹화에 최적화되어있음

4.  Q n A 및 회의록 기록

groupingBy에서도 미리 선언해놓은 Map을 전달할 수 있나요?
네 가능합니다 -> MapFactory에 참조가 가능하고 merge과정도 알아보면 좋을 것 같습니다 (어떻게 최적화하는지?)

---

## 이슈7 [스트림의 게으른 계산의 장점]

1. 안건을 올린 이유 발표
- 스트림의 가장 큰 장점인 것 같은데 이 기회에 알고가면 좋을 것 같아서 골랐습니다.
  
2. 안건에 대해 답변자 발표 및 설명 요약
- 늦은 계산법 : 중간 연산만 있을 땐, 연산 x, 최종 연산 있을때야 연산
- 효과 :
  - 미래를 알고 있기 때문에 계획을 최적화가 가능하다. (불필요한 데이터를 줄일 수 있음)
  - 무한 데이터 처리 가능
  - 중간에 유연한 변경과 조합이 가능
- 백트래킹과 같은 느낌으로 생각 (가지치기)

  
3.  Q n A 및 회의록 기록
- 이전 값이 필요하다면?
  - sort, reduce 같은 연산이 필요하다면 가지치기 x
- 순서가 필요?
  - 상황에 따라 다름


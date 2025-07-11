## DAY_12(Mon)
- [제네릭](#제네릭)
- [FILE I/O](#파일I/O)
- [Big O](#Big-O)

---

### 제네릭
<T> 타입 매개변수 ( 타입 매개변수 ) <br>
원시(기본)자료형 사용 불가 (int, double...) <br>
참조자료형 -> heqp에 있는 주소값만 가리킴 <br>
원시자료형 -> stack에 그대로 저장 <br>

call by value = 값을 전달 ( 주소를 가지고 있는 참조자료형은 주소를 줌 )

자바 제네릭
- 타입을 미리 구체적으로 지정하지 않음
- 사용하는 시점에 사용자가 결정
- 특정 데이터 타입에 얾매이지 않도록 하는 것

제네릭 클래스와 제네릭 메서드는 다를 수 있음

#### 제네릭 클래스
**대표적으로 collection**<br>
T type / E element / K key / V value <br>
static (클래스 변수)에 타입 매개변수 사용 불가 <br>
- 클래스 변수를 통해 동일한 변수를 공유하는 것이 아니게 됨 ( 동일한 클래스에서 하나는 Integet, 하나는 String일 경우 공유 불가 )
참조 자료형만 가능 int -> Integer, double -> Double <br>
다형성 적용 가능 ( Flower 타입과 Flower타입을 상속받은 Rose가 있을 시 Flower 타입의 제네릭 클래스에 Rose 사용 가능 ) <br>

#### 제한된 제네릭 클래스
타입 매개변수 T를 제한 함 ( T extends Flower )
- 다형성 적용이 가능하기 때문에 가능 함 ( (T extends 상위클래스)로 선언해서 하위클래스만 허용 )
- -> 상위 클래스로 형변환이 가능한 것만 ( 상위 클래스 포함 )
- 인터페이스를 상속받는 타입만 허용하고 싶을 때도 extends 사용 ( T extneds 인터페이스 명 )
- 클래스와 인터페이스 동시 제한 가능
- -> & 연산자 사용 ( T extends 상위클래스 & 인터페이스 명 )

#### 제네릭 메서드
**클래스 내부의 특정 메서드만 제네릭으로 선언 가능** <br>
**제네릭 메서드의의 타입 매개 변수는 제네릭 클래스 타입의 매개 변수와 별개의 것** ( 동시에 사용하더라도 *서로 다른 타입으로 간주* ) <br>
- **클래스** 타입 파라미터는 **인스턴스화 시점**에 지정
- **메서드** 타입 파라미터는 **메서드 호출 시점**에 지정
- ***static 메서드로 선언 가능***
- -> 하위 메서드 사용 불가 (.length()...) BUT, Object 메서드는 사용 가능 ( 최상위 클래스로써 형변환 가능 )
- 제네릭 타입은 자동으로 유추 가능 함 ( 지정 안해도 됨 )

#### 와일드카드(<?>)
**모호한 타입 또는 범위를 제한하고자 할 때 사용**
- <? extends T> 상한 제한 / 읽기 전용 / T 및 그 하위 타입 허용
- <? super T> 하한 제한 / 쓰기 전용 / T 및 그 상위 타입 허용
- 상한 제한의 경우 하위 클래스의 값이 상위 클래스에 적합하다고 볼 수 없음 (Number -> Integer, Double) / Number의 값이 정수(Integer)일 지 실수(Double)일 지 알 수 없음
- -> 읽기 전용
- 하한 제한의 경우 읽는 값이 상위클래스의 적합하다고 볼 수 없음 (Number가 object일지 Number일지 알 수 없음) / Number의 값이 object일 수 있음 (다운캐스팅 불가의 경우)
- -> 쓰기 전용
- **제네릭 객체를 사용하는 쪽(매개변수 등)에서 사용**

- 제네릭 메서드
- > public <T> void call(phone){};
- 와일드카드
- class iPhone<T> 로 타입 매개변수 지정 시 ( 반환 타입을 제한할 때 )
- > public void call(iPhone<? extends Iphone>iphone){};

#### 래퍼클래스
기본타입 데이터를 인스턴스화 하여야 할 때 사용 ( int -> Integer, double -> Double... )<br>
boxing : 기본타입 -> 객체<br>
Unboxing : 객체 -> 기본타입<br>
*null 가능*<br>
정수의 캐싱범위 : Byte ( -128 ~ 127 ) Integer도 Long도 Short도 해당 범위
- 캐싱 범위 내에서는 동일한 값일 시 a == b true BUT, 캐싱 범위 밖일 시 동일한 값이더라도 false
- 캐싱 : 미리 만들어둔 값을 공유
- -> .equals()로 값 비교 권장
- 문자열 -> 기본 타입 ( Integer.parseInt("123"); )
- 기본타입 -> 문자열 (Integer.toString(); / 123+"" : 문자열 결합으로 문자열로 변환)

#### 제네릭 컬렉션 래퍼
타입 안정성 보장 방법
- List<T>
- 타입 매개변수 명시
- 경계 와일드카드 활용
- 메네릭 유틸리티 메서드

---

### 파일I/O
절대 경로(Absolute Path) : 루트 디렉토리부터 시작하는 전체 경로 <br>
상대 경로(Relative Path) : 현재 작업 디렉토리 기준으로 표시 ( ./ 현재 디렉토리 , ../ 상위 디렉토리 )

자바에서는 File 클래스를 통해 파일 및 디렉터리를 추상화하여 다룰 수 있음

입출력 스트림은 **단방향**으로 동작 / 입력 스트림과 출력 스트림으로 나뉘어짐

스트림 기반 입출력
- 바이트 기반 스트림
  - InputStream, OutputStream
  - 영상이나 이미지 처리에 활용
- 문자 기반 스트림
  - Reader, Writer
  - 문자열 처리에 활용 ( 문자열, 정수, 실수 등 문자열로 표현 가능한 것 )

InputStrema
- FileInputStream은 try-catch 필수 -> 생성자에서 예외를 throw 하고 있음 -> 받아서 쓸라면 예외처리를 받아줘야 함
- 보조 스트림 (BufferedInputStream) 내부적으로 버퍼를 사용하여 읽기 성능 향상
- -> 버퍼를 채우고 버퍼가 가득 차면 그때 동작 / 사용안하면 읽을 때 마다 동작함

OutputStream
- **기존 파일이 있으면 덮어쓰기 없으면 새로 생성이 기본**

자료구조 데이터의 영속화
- 파일로 저장하고 관리하는 것
- 자바는 유니코드 기반 / 각 인코딩과 유니코드 간 변환을 자동으로 처리
- 스트림 인스턴스를 생성할 때 예외처리를 수행할 수 있도록 **try-catch 구문 사용**
- **스트림을 닫지 않으면** 메모리에 상주하면서 리소스를 잡아먹기 때문에 **리소스 누수가 발생** -> close()나 flush()로 스트림을 닫아줘야함
- br.flush() = 버퍼를 비우고 버퍼 내용을 처리

---

## Big O
알고리즘과 프로그램의 성능을 정량적으로 분석 <br>
- 입력 크기에 따른 연산 횟수 증가율에 초점 -> OS와 상관 없음
- 실행 시간 뿐 아니라 메모리 사용량도 분석 가능 ( 시간복잡도 , 공간복잡도 )
- 최악의 경우 고려
- ***점근적 분석***
  - 입력 크기 증가에 따른 성능 변화 예측
  - 시스템이 성장했을 때의 대응 능력을 판단하기 효과적
  - 단위 시간의 차이에 집중하기보다 **알고리즘의 구조적 차이**(O(n),O(n log n))를 더 중요하게 생각한다.
  - 객관적 지표
- 계수는 고려하지 않음
- O(2^n) > O(n^2) > O(n) > O(log n) > O(1) / 걸리는 시간
 
#### 점근적 표기법
- 차수 - Big-Theta
  - 평균을 산정
- 상한 - Big-O
  - 최악의 경우
- 하한 - Big-Omega
  - 최선의 경우

#### 주요 시간 복잡도
- O(1) 상수 시간
  - 입력 크기에 상관없이 항상 일정한 시간 내에 연산이 완료되는 경우
  - 배열에서 특정 인덱스 값 조회
  - **HashMap에서 키로 값 조회**, stack의 peek(),push(),pop()
- O(log n) 로그 시간
  - 매 연산마다 입력을 절반으로 줄이는 방식 / **이진 탐색**
  - binary search, 이진 탐색 트리, heap의 삽입 및 삭제
- O(n) 선형 시간
  - 입력의 크기에 비례
  - 단순 반복문, 배열검색, List 전체 순회
- O(n log n) 선형 로그 시간
  - 재귀적으로 반으로 나눈 뒤, 각 부분에 대해 n번 처리
  - 정렬에 사용 (퀵 정렬, 병합 정렬)
  - merge Sort, Quick Sort (평균/최선)
  - heap Sort
  - 효율적인 정렬 알고리즘 전반
  - Java의 Arrays.sort()
- O(n^2) 제곱 시간
  - 중첩 반복문
  - 버블 정렬, 삽입 정렬
  - bubble Sort, insertion Sort, Selection Sort
  - 인접 행렬 기반 그래프 탐색 / DFS,BFS
  - 중복 비교가 필요한 경우 (모든 조합 찾기 등) / 조합
  - Brute Force 방식 알고리즘

#### 공간 복잡도
작업을 수행하는 동안 **사용하는 메모리의 양** / 입력 데이터 저장공간 + 임시 작업 공간 <br>
효율적인 메모리 사용은 성능 최적화의 핵심
- 보조 공간
  - 입력 데이터를 제외하고, 알고리즘이 **추가적으로 사용하는 공간**
  - in-place : 입력 데이터만으로 작업을 수행하는 공간 O(1)
  - non-in-place : 별도의 자료구조를 사용 O(n)
- trade-off
  - 하나의 가치를 얻기 위해 다른 가치를 포기하는 선택 (반비례)
  - 시간과 공간은 반비례 관계이다
  - 캐싱과 메모이제이션 기법이 대표적 / 추가 공간을 사용해 속도를 빠르게 한다 / DP 알고리즘
  - 스트리밍의 경우 속도를 느리게하는 대신 공간을 적게 사용
  - 

#### 선형 자료구조 Big-O
- ArrayList
  - Access O(1) / Search O(n) / insertion O(n) / Deletion O(n)
  - CPU 캐시 효율이 높음
- LinkedList
  - Access O(n) / Search O(n) / insertion O(1) / Deletion O(1)
  - 메모리 접근 비용이 크다
- Stack
  - Access O(n) / Search O(n) / insertion O(1) / Deletion O(1)
  - peek 조회는 O(1)
- Queue
  - Access O(n) / Search O(n) / insertion O(1) / Deletion O(1)
  - peek 조회는 O(1)
 
- Deque
  - ArrayDeque
    - 삽입/삭제 O(1) BUT 확장 시 내부 배열 복사 O(n)
    - 제한된 크기, 확장시 내부 배열 복사 필요 ( O(n) )
  - LinkedList
    - 유연한 구조로 확장 가능 ( O(1) )
   
#### 비선형 자료구조 Big-O
- Tree
  - 균형 이진 트리 깊이 log n / O(log n)
  - 서브트리의 높이 차이가 일정 범위 내로 유지되는 트리 / O(log n) 보장 / Red-Black-Tree
  - 불균형 이진 트리 깊이 n / O(n)
  - 한쪽으로 노드가 몰려 있는 형태 / 사실상 리스트 처럼 동작 O(n)으로 퇴화
  - 삽입 삭제 검색 O(log n) / 순회 (중위 순회 기준) O(n)
  - 항상 정렬된 상태 유지 / 범위 검색이나 정렬 기반 탐색에는 강력
- HASH
  - 해시 충돌이 많아지면 리스트나 트리로 바뀌어 성능이 O(n)까지 떨어 질 수 있음
  - 평균 검색, 삽입, 삭제 O(1) / 최악 검색, 삽입, 삭제 O(n)
  - 충돌이 많아지면 내부 구조를 Red-Black Tree로 전환

#### 자료구조 선택 가이드
- 소규모 (n<=100)
  - 속도보다는 코드 간결성과 가독성 / 아무거나 써도 될 정도
  - 일반적으로 **ArrayList**, LinkedList, Stack 등 단순 구조
- 중간 규모 (수천~수만)
  - 연산의 특성( 탐색 vs (중간)삽입/삭제 vs 정렬 ) 에 따라 선택이 갈림
  - 정렬 = TreeMap, TreeSet
  - 빠른 탐색 = HashMap, HashSet

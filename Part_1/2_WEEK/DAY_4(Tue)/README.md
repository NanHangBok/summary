# DAY_4

- [복습](#복습)
- [SOLID 원칙](https://github.com/NanHangBok/summary/edit/main/Part_1/2_WEEK/DAY_4(Tue)/README.md#solid-%EC%9B%90%EC%B9%99)
- [내부/익명 클래스](#내부-클래스와-익명-클래스)
- [Java Collection Framework](#JCF(Java-Collection-Framework))
  - [Collection 모음](#컬렉션)
- [람다 표현식](#람다-표현식)
## 복습

#### 상속

- 확장
- super, super()
- 단일 상속
- 포함 관계
- 오버라이딩 ( 재정의 ) (override)
- extends

#### 캡슐화

- getter / setter
- 접근제어자 (public / protected / default / private)

#### 다형성

- 오버로딩 / 오버라이딩
- 형 변환 (업캐스팅/다운캐스팅)
- instanceof
- 업캐스팅 -> 자동 / 다운캐스팅 -> 강제 형변환

#### 추상화

- 추상클래스
- 추상메서드
- 인터페이스
  - 다중 구현
  - 객체 생성 불가
  - implements
  - default 메서드 지원
  - 인터페이스 extends 인터페이스

#### 상속 & 추상화

- 상속은 데이터 중심 설계
- 인터페이스는 기능 중심 설계

---

## SOLID 원칙
**객체지향 설계에서 5가지 설계 원칙**

##### 단일 책임의 원칙 (SRP)
> 하나의 클래스는 하나의 책임만 가져야 한다
##### 개방/폐쇄의 원칙 (OCP)
> 확장에는 열려있고, 수정에는 닫혀있어야 한다
##### 리스코브 치환의 원칙 (LSP)
> 자식 클래스는 반드시 부모 클래스를 대체할 수 있어야 한다
##### 인터페이스 분리의 원칙 (ISP)
> 하나의 범용 인터페이스보다 다수의 구체적인 인터페이스가 낫다
> 클라이언트는 사용하지 않는 메소드에 의존하면 안 된다
##### ****의존성 역전의 원칙 (DIP)***
> 구체적인 클래스 보다는 인터페이스나 추상 클래스와 관계를 맺어야 한다
> 상위 모듈은 하위 모듈에 의존하면 안된다
> 추상화에 의존해야지, 구체화에 의존하면 안된다

#### 단일 책임 원칙
- 클래스는 하나의 책임만 가져야 한다.
책임이란 역할이며 변경의 사유가 된다 -> A라는 클래스가 2가지 이상의 역할을 수행한다. (입출력 / 입력과 출력 분리)

기능별로 분리 / 관심사(UI/로직/저장소 등)에 따라 분리 ( 계층형 구조 ) / 분리된 클래스(인터페이스)는 하나의 책임만 갖도록 설계

#### 개방/폐쇄 원칙
- 기존 코드를 변경하지 않고, 새로운 기능은 확장할 수 있도록 설계
- 인터페이스를 통해 구현체가 무엇이든 상관하지 않고 메서드를 사용할 수 있도록 한다
  - 인터페이스의 의존하는 클래스의 수정이 필요없어진다.
- 런타임에 전략 주입 -> 생성자를 통해 객체가 생성될 때 인터페이스의 하위 클래스를 인터페이스 객체로 만들어 주입한다 (생성자 주입)
 
#### 리스코브 치환 원칙
- 상위 타입으로 하위 타입을 사용해도 프로그램의 동작에 문제가 없어야 한다
1. 계약의 의한 설계
  - 하위 클래스가 부모의 메소드를 재정의하더라도, 기존 계약을 위반해서는 안됨
  - 사전조건
    - 메소드 실행 전에 만족해야 할 조건
    - 제약 조건을 메소드가 실행되기 전에 실행 ( 상위 클래스에서 )
  - 사후조건
    - 메소드 실행 후 반드시 만족해야 하는 결과 ( 타입 등 )

LSP 전제 조건 = 다형성 (상속 & 구현)<br>
~오버라이딩 시 기능을 제거하거나 조건 강화~ -> 위반

#### 인터페이스 분리 원칙
- 하나의 커다란 인터페이스보다는 작고 구체적인 인터페이스 여러개가 좋다
- 작고 응집도 높은 인터페이스를 여러개 만드는 설계

#### 의존성 역전 원칙
- 구체적인 클래스에 의존하지 않고 인터페이스(추상화)에 의존
- **의존 방향을 뒤집는다**
- 인터페이스 분리 + 의존성 주입으로 구현
- 고수준 모듈 : 핵심 비즈니스 로직 (추상화/인터페이스)
- 저수준 모듈 : 세부 구현 로직 (구현클래스)
- 의존성 주입
  - 외부로부터 주입받는 방식 ( 구현체가 직접 생성하지 않음 )
  - **생성자 주입** , 세터 주입 , 필드 주입
  - 생성자 주입이 가장 일반적 / null 방지

---

## 내부 클래스와 익명 클래스

#### 내부 클래스
클래스 내부에 선언된 또 다른 클래스 (인스턴스 내부 클래스)
- 이벤트 리스너 -> 외부 클래스와 밀접한 기능 (강하게 연관된 관계)
- private, protected로 선언하여 외부에서 접근이 불가능하게 만듬
- 인스턴스 내부 클래스 - 외부 인스턴스 변수 접근 가능
- static 내부 클래스 - DTO
- 지역 내부 클래스 - 메서드 내에서 사용
- 익명 내부 클래스 - 일회성 클래스

#### 멤버 내부 클래스
- 인스턴스 내부 클래스
  - 외부 클래스의 인스턴스가 있어야 생성 가능 ( Outer o = new Outer; 가능 / Outer.Inner i = new Outer.Inner(); 불가능 )
  - 외부 클래스의 모든 멤버(인스턴스 변수/static 변수)에 접근 가능
- 정적 내부 클래스
  - 외부 클래스의 인스턴스 없이도 생성 가능 ( Outer.Inner() inner = new Outer.Inner() 가능 )
  - 여러개의 외부클래스가 하나의 이너클래스를 공유
  - 외부 클래스의 static 멤버(static 필드, static 메서드)에만 접근 가능
- 멤버 내부 클래스는 외부 클래스와 논리적으로 밀접한 관계일 때 유용 ( 응집도 높은 설계 가능, 캡슐화에 도움 )

#### 지역 내부 클래스
특정 클래스 안에 메서드 내부의 클래스<br>
**메서드의 지역 변수는 반드시 final이거나 effectivley final(실질적 상수)이여야 한다**<br>
effectivley final = 지역내부 클래스 or stream / 지역 변수에 final 제어자 추가

#### 익명 클래스
*람다식과 관계있음*,*람다 + 함수식 인터페이스*<br>
일회용으로 이벤트 처리나 콜백 처리 등에서 활용 -> 재활용하지 않음(한번만 사용함) <br>

---

## JCF(Java Collection Framework)
- 배열의 한계점
  - 고정 크기 (데이터의 수가 동적일 때 다루기 불편)
  - 타입 안정성 문제 (ClassCastException)
- JCF로 단점 보안
  - 동적 크기 ( 자동으로 크기 조절(확장) )
  - 타입 안정성(제네릭) / List<String> 등으로 컴파일 타임에 배열에 들어갈 타입 검사 가능
  - 편의 기능 제공 ( add(), remove(), size() 등 기본 메서드 제공) / sort(), distinct(), filter() 등 스트림과 함께 정렬/중복 제거 가능
  - 제네릭은 참조자료형만 가능 / 원시(기본)자료형은 wrapper 클래스로 사용 (ex: int -> Integer)
  - List, Set, Map 등 다양한 자료 구조 지원
- framework vs library
  - 제어권의 차이
  - framework이 가지고 있음 -> 시키는대로(정해진 사용법대로) 사용한다
- 컬렉션 : 데이터의 집합 -> 효과적으로 다루기 위한 도구 모음 : 컬렉션 프레임워크

#### JCF
1. 사용하는 이유
   - 일관된 API 제공 ( 동일한 메서드 사용 )
   - 프로그래밍 생산성 향상 ( 구현된 클래스 사용 )
2. 구조 요약
   - Collections + Map
   - (인터페이스)List = **LinkedList** & Stack & Vector & **ArrayList**
3. 주요 인터페이스
   - List (**순서 유지**, 중복 허용) / **ArrayList**, LinkedList / 순회 가능
   - Set (순서 없음, **중복 불가**) / HashSet, TreeSet / 순회 가능
   - Queue (**FIFO**, 순차처리) / **LinkedList**, PriorityQueue
   - Deque (양방향 Queue, 스택 + 큐) / *ArrayDeque*, LinkedList
   - Map (**키-값 쌍**, 키 중복 불가) / **HashMap**, TreeMap / Collection 계열이 아닌 별도 인터페이스
4. 컬렉션 간 변환 관계
   - List -> Set = 중복제거
   - Set -> List = 순서부여
   - Set -> Map = 요소를 키로 값은 가공된 결과로 저장
   - **Map -> Set = keySet(), entrySet() 이용** / entry = K-V 1:1 매핑
   - List -> Map = 특정기준 (인덱스,ID)을 key로 가공
- 정리
  - 데이터 구조
    - collection : 단일 값 집합
    - map : key-value 쌍
  - 주요 인터페이스
    - collection : List, Set, Queue, Deque
    - map : Map
  - 중복 허용
    - collection : List만 허용
    - map : key X, value O
  - 순서 보장
    - collection : List, 일부 Queue
    - map : LinkedHashMap
  - 주요 목적
    - collection : 값의 모음
    - map : key 기반 탐색

### 컬렉션

#### ArrayList
배열 기반 구조, 인덱스를 통한 빠른 검색<br>
제네릭을 앞에 사용하기 때문에 **인스턴스 생성 시 생략 가능** <br>
중간 삽입/삭제의 경우 배열 이동이 필요 -> 느릴 수 있음

#### LinkedList
연결 리스트 자료구조 기반 <br>
삽입/삭제가 빠르고, 검색이 느림 <br>
각 요소의 이전/다음 요소의 참조 주소를 저장하고 있음 <br>
**중간에 삽입/삭제가 빈번하게 일어날 때 효과적**

#### SET
중복 X, 순서보장 X (인덱스 없음), 순회 O

#### HashSet
해시 테이블 구조 사용 <br>
객체 저장 시 hashCode()로 해시값 생성 <br>
동일 해시 값 존재 시 *equals()*로 추가 여부 판단
- **서로 다른 객체의 특정 멤버를 set으로 관리하고 싶다면 equals()와 hashCode()를 오버라이딩 한다.**

#### MAP
**Key 중복 X, Value 중복 O** <br>
Key 기반 데이터 저장 및 검색 -> **검색 속도가 빠름** <br>
map.put() = 중복 Key 시, 덮어씌움 / map.putIfAbsent() = 중복 key 시, 추가(덮어씌움) X <br>
map.get() 으로 없는 값을 가져올 경우 null 반환 -> *containsKey, containsValue 사용하기*
- 순회 시 keySet(), entrySet() 활용

#### HashMap
해싱 기법을 통해 key를 빠르게 검색 <br>
저장 순서 유지 X
- Entry 객체 : map 내부에 저장되는 한 쌍의 데이터 (key, value)

#### 컬렉션별 적용 사례
1. 학생 명단 관리 (순서 유지 + 중복 허용) List
2. 출석 체크 기록 (중복 제거) Set / 유니크한 데이터 일 것
3. 학생 이름과 점수 매핑 (Key-Value) Map
   - 확장
   - ```
     Map<String, Map<String, Integer>> exam_scores = new HashMap<>();
     Map<String, Integer> scores = new HashMap<>();
     socres.put("korean",100);
     scores.put("eng",80);
     exam_scores.put("name",scores);
     ```

- Set -> List 정렬
- List -> Set 중복제거
- List -> Map key=name, value=count
- ~Map -> List~ 는 불가능

---

## 람다 표현식
***메서드를 하나의 식으로 표현한것 (메서드를 다른방식(표현식)으로 변경)*** <br>
**인터페이스 객체를 만들어 인터페이스에 정의된 메서드의 내용을 만든다** <br>
메서드 이름과 리턴 값 없음 (익명 함수) <br>
> 함수 이름을 빼고 -> 추가 <br>
> void print() = { System.out.println("hello");} --> **() -> System.out.println("hello");**

```
interface Add {
  int sum(int a, int b);
}

Add add = (a,b) -> a+b;
add.sum(1,2);

interface Print {
  void show();
}
Print p = () -> System.out.println("람다식");
p.show();
```

메서드의 파라미터로 메서드를 사용하기 위해 람다표현식 사용 / *오직 람다만 가능*
- 반복적인 문법을 간결하게 줄일 수 있다
- 많은 API가 람다 기반 설계

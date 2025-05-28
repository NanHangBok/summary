# DAY_5

- [람다식](#람다식)

## 람다식
```
//기존 코드
public interface Calc {
  int sumTwoNum(int a, int b);
}

public class CalcClass implements Calc {
  @Override
  public int sumTwoNum(int a, int b) {
    return a+b;
  }
}

public Main {
  public static void main(String[] args) {
// 클래스 구현, 인스턴스 생성 후 메서드 호출
    Calc c = new CalcClass();
    System.out.println(c.sumTwoNum(5,10));
// or
    // 익명 클래스
    Calc c2 = new Calc() {
      @Override
      public int sumTwoNum(int a, int b) {
        return a+b;
      }
    };
    System.out.println(c2.sumTwoNum(1,2));

// 람다식표현
    Calc c3 = (a,b) -> a+b;
    System.out.println(c3.sumTwoNum(2,4));
}
```

#### 축약 문법
- 매개변수 0개 : () -> System.out.println("람다식");
- 매개변수 1개
  - (String name) -> System.out.println(name);
  - (name) -> System.out.println(name);
  - **name -> System.out.println(name);**
  - 3개다 가능
- 매개변수 2개 : () 필수 / sum = (a,b) -> a+b;
- 바디가 2줄 이상
  ```
  name -> {
    String result = "hi"+name;
    return result
  };
  ```
- 바디가 1줄
  ```
  x -> { return x*x; }; 가능
  x -> { x*x; }; 불가능
  x -> x * x; 가능
  // {}와 return은 동시에 존재해야 한다
  ```
#### 변수 캡처
람다식 내부에서 외부 지역 변수(로컬변수)를 참조 가능 but **그 변수는 반드시 final or effective final**

#### 함수형 인터페이스
**추상 메서드가 1개만 있는 인터페이스** / Object 클래스에서 상속받은 메서드(toString(),equals())는 포함되지 않는다<br>
**@FunctionalInterface 어노테이션 사용**

#### 함수 디스크립터
매개변수 타입, 개수, 리턴 타입을 묶어 부르는 것 -> 람다식이 이 시그니처와 맞아야 유효한 할당

#### 주요 함수형 인터페이스
- Consumer 계열
  - 매개변수 O, 리턴값 void
  - (T) -> void
  - Consumer void accept(T t)
  - BiConsumer<T, U> void accept(T t,U u)
- Supplier 계열
  - 매개변수 X, 리턴값 O (리턴값만 존재)
  - () -> T
- Function 계열
  - 매개변수 O, 리턴값 O
  - (T) -> R
  - Function<T,R> R apply(T t)
  - BiFunction<T,U,R> R apply(T t, U u)
 - Operator 계열
   - Function의 특수화
   - 입력값과 리턴값이 동일
   - (T) -> T
   - UnaryOperator T apply(T t)
  - Predicate 계열
    - 리턴 타입 boolean
    - (T) -> boolean
    - Predicate boolean test(T t)
    - BiPredicate boolean test(T t, U u)

#### 메서드 참조
람다 표현식의 축약형/ :: 연산자 사용
- 정적 메소드 참조
  - 클래스명::메서드명
  - static 메서드를 참조하는 방식
- 특정 객체의 인스턴스 메서드 참조
  - 참조변수::메서드명 ( 변수명::메서드명 )
  - 특정 객체의 인스턴스 메서드 참조
- 임의 객체의 인스턴스 메서드 참조
  - 클래스명::메서드명
  - 임의 객체의 인스턴스 메서드 참조
  - 첫 번째 매개변수가 메서드 호출의 주체
  - 첫 번째 매개변수 클래스(타입)의 메서드를 호출한다 BiFunction<String,String,boolean> fnc = String::equals
- 생성자 참조
  - 클래스명:new
  - 객체 생성만 하는 람다식
  - 생성자 오버로딩에 대응 가능

## Stream
**배열, 컬렉션의 저장 요소를 하나씩 참조해서 람다식으로 처리할 수 있도록 해주는 반복자** <br>
List, Set, Map, 배열 등 다양한 데이터 소스로부터 스트림을 만들 수 있음 (표준화된 방법이 있음) <br>
  데이터 소스에 상관없이 같은 방식으로 가공/처리 가능 / 가독성 증가 <br>
순회자 List -> Stream -> 순회 <br>
대용량 처리 ( 대용량 처리 시 for문 보다 빠름 ) <br>

- 특징
  - 스트림 처리과정 (생성 -> 중간 연산 -> 최종 연산 ) 3단계의 파이프라인으로 구성
  - 스트림은 원본 데이터 소스를 변경하지 않음 (READ ONLY)
  - 스트림은 일회용 (Onetime-Only)
  - 스트림은 내부 반복자
 
- 스트림 파이프라인
  - 생성 / stream으로 변환
  - 중간 연산 / 0번 이상 가능
  - 최종 연산 / 단 한번만 가능

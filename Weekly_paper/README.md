# WEEKLY_PAPER
- [2주차](#2주차) [rebase와 merge / fetch와 pull]
- [3주차](#3주차) [단일책임원칙(SRP)와 개방-폐쇄원칙(OCP) / stream 연산자 map()과 flatMap()]
- [4주차](#4주차) [해시셋 / O(n)과 O(log n)]
- [5주차](#5주차) [Spring Framework 등장 배경 / Framework vs Library]
## 2주차

+ git rebase와 git merge의 차이점을 설명하고, 각각 어떤 상황에서 사용하는 것이 적절한지 설명해주세요.
  + **rebase**의 경우 히스토리가 남지 않고 **히스토리가 선형**이 되고 개인 브랜치에서 추가 브랜치를 만들어 개인적으로 사용할 때 보통 사용하고,
  + **merge**의 경우 히스토리가 남는다.
  + *( 개인적인 생각으로 기업에서 버전관리를 중요시한다면 불필요한 경우 merge를 사용하여 히스토리를 남겨 버전 업데이트 내역에 대한 가독성을 떨어트리는 것 보다 rebase를 통해 히스토리를 남기지 않고 병합하는 방식이 적합할 것 같다.)*


+ git fetch와 git pull의 차이점을 설명하고, 각각을 사용하는 것이 적절한 상황을 설명해주세요.
  + fetch의 경우 **local repository로 데이터를 가져오기** 때문에 working directory를 건들지 않기 때문에 현재 작업중인 내용이 수정되지 않기 때문에 변경사항을 확인할 때 사용할 수 있고 
  + pull의 경우 **remote repository에서 working directory로** 가져오기 때문에 자신의 **working directroy(local repository)와 remote repository를 연결**할 수 있고 현재 working directory의 **작업이 덮어 씌워지며** 작업중인 것이 유실 될 수 있다.

---

## 3주차

+ 객체지향 프로그래밍에서 '단일 책임 원칙(SRP)'과 '개방-폐쇄 원칙(OCP)'에 대해 설명하고, 각각의 원칙을 적용한 코드 예시를 들어주세요.
  + **단일 책임 원칙(SRP)**은 하나의 클래스는 하나의 기능만 가지고 있어야 한다는 객체지향 설계 원칙이며
    + **기능별로 분리, 관심사의 분리, 하나의 책임만 갖도록 설계**하여 적용할 수 있다.
    + ```
      class Report {
          String title;
          String content;

          void saveToFile() { ... }
          void print() { ... }
      }    // 저장과 출력 2가지 기능(책임)을 하고 있다.
      // ----------------------------------------------------------
      class Report {
          String title;
          String content;
      }
      class ReportPrinter {
          void print(Report report) {
              ...
          }
      }
      class ReportSaver {
          void saveToFile(Report report) {
              ...
          }
      }    // report라는 객체를 출력하고 저장하는 클래스를 따로 사용한다.
      ```
  + **개방-폐쇄 원칙(OCP)는 **기존 코드를 변경하지 않고, 새로운 기능은 확장을 통해 추가**할 수 있도록 설계하는 설계 원칙이다.
    + ***핵심 인터페이스 정의*** -> 다양한 구현체 -> 런타임에 전략 주입은 OCP에 아주 적합한 구조이다.
    + ```
      class PaymentService {
          void pay(String method) {
              if (method.equals("card")) {
                  sout ...
              else if (method.equals("kakao")) {
                  sout ...
              }
          }
      }    // 새로운 결제 수단이 생길때마다 pay() 메소드를 수정해야 함
      // ---------------------------------------------------------
      inter face PayStrategy {
          void pay();
      }

      class CardPay implements PayStrategy {
          public void pay() {
              sout ...
          }
      }
      class KakaoPay implements PayStrategy {
          pbulic void pay() {
              sout...
          }
      }
      class PaymentService {
          private PayStrategy payStrategy;
          public PaymentService(PayStrategy payStrategy) {
              this.payStrategy = payStrategy;
          }
          public void precessPayment() {
              payStrategy.pay();
          }
      }    // 새로운 결제 수단이 생기면 새 클래스를 생성하여 확장 가능
      // 기존의 PaymentService 클래스는 수정하지 않아도 됨-------------
      ```
      
+ Stream API의 map과 flatMap의 차이점을 설명하고, 각각의 활용 사례를 예시 코드와 함께 설명해주세요.
  + map은 **원하는 필드만 추출하거나 특정 형태로 변환**할 때 사용되는 중간 연산자이다.
    + **스트림의 요소 변환은 원본 객체에 영향을 주지 않는다.**
    + ```
      import java.util.Arrays;
      import java.util.List;
      
      public class IntermediateOperationExample {
          public static void main(String[] args) {
              List<Integer> list = Arrays.asList(1, 3, 6, 9);

              // 각 요소에 3을 곱한 값을 반환
              list.stream().map(number -> number * 3).forEach(System.out::println);
          }
      }
      
      // 출력값
      3
      9
      18
      27
      ```
  + flatMap()은 **중첩된 스트림 구조를 평탄화하여 하나의 스트림으로 펼쳐주는 연산자**이다.
    + *이중 이상의 리스트*의 경우 하나의 **자료를 컨트롤하기 복잡**하기 때문에 이 flatMap을 사용하여 **평탄화를** 한 뒤 컨트롤 하기도 한다. ***(1차원의 형태로 가공한다)***
    + ```
      List<List<String>> nestedList = Arrays.asList(
          Arrays.asList("JAVA", "SPRING"),
          Arrays.asList("java", "spring")
      );
      /*
      	기존 리스트의 형태
      [
      	["JAVA", "SPRING"],
      	["java", "spring"]
      ]
      */
      nestedList.get(0).get(1) -> "SPRING"
      nestedList.stream()
          .flatMap(Collection::stream) // 기존의 stream을 Collection의 stream의 형태으로 flat하게 만들겠다.
          //-> ["JAVA", "SPRING", "java", "spring"]  1차원의 형태로 가공하겠다.
          .forEach(System.out::println);
      ```

---

## 4주차
+ HashSet의 내부 동작 방식과 중복 제거 메커니즘을 설명하고, HashSet이 효율적인 중복 체크를 할 수 있는 이유를 설명해주세요.
  + 내부적으로 hashMap을 사용 -> **key로 원소**를 사용하여 **value는 모두 동일한 고정 객체**(PRESENT)를 사용
  + 원소만 저장 -> hash function을 이용하여 입력 값을 고정된 크기의 정수(해시값)로 변환하는 함수 -> 해시값은 배열의 인덱스로 사용
  + 데이터를 빠르게 저장하거나 검색 가능
  + 여기서 HASH값이 VALUE(원소)값
  + 동일한 HASHCODE 발생 시 equals로 동일 객체인지 확인 -> 다른 객체 일 시 새로운 원소로 저장 -> **충돌방지 중 체이닝을 통해 연결리스트로 저장**
  + 원소를 KEY로 사용하기 때문에 서로 다른 객체가 동일한 HASHCODE를 가지게 될 수 있는데 이 때, **equals()를 통해 동일한 객체인지 판단**
  + 서로 다른 객체 일 시 **충돌방지를 통해 해당 객체를 새롭게 저장**(체이닝)한다.

+ O(n)과 O(log n)의 성능 차이를 실생활 예시를 들어 설명하고, 데이터의 크기가 1백만 개일 때 각각 대략 몇 번의 연산이 필요한지 비교해주세요.
  + O(n)은 리스트 전체 순회이다. 리스트 안의 **요소의 개수만큼 연산이 진행**된다.
  + 번호가 적힌 공이 담긴 주머니에서 내가 원하는 숫자를 뽑을 때까지 반복하는 것과 비슷하다. **최악의 경우 모든 공을 꺼내어 마지막 공이 내가 원하는 숫자**일 수 있다.
  + O(n)은 데이터의 크기가 1백만 개일 때 ***1백만번의 연산***을 실행한다.
  + O(log n)은 이진 탐색과 비슷하다. **데이터 개수를 절반으로 나누면서 1이 될 때까지 연산**한다.
  + O(log n)은 실생활의 업-다운 게임과 비슷하다. **수의 중간을 부르면서 절반씩 줄여나가**면 최악의 경우 가장 빠르게 정답을 알 수 있다.
  + O(log n)은 데이터의 크기가 1백만 개 일 때 ***20번의 연산***이 필요하다.

---

## 5주차
+ Spring Framework가 탄생하게 된 배경과 이를 통해 해결하고자 했던 문제점에 대해 설명하세요.
  + 기존 *EJB 중심은 복잡한 설정, 높은 기술 종속성, 테스트의 어려움 등으로 인해 한계*가 있음.
  + EJB를 대체할 수 있는 경량 프레임워크의 필요성의 필요성
  + **POJO 기반 개발** / 컨테이너에 종속되지 않는 순수한 자바 객체 사용
  + **IoC(Inversion of Control)** 객체의 생성과 의존성 주입을 개발자가 아닌 컨테이너가 담당
  + **AOP(aspect-Oriented Programming)** / 공통 관심사를 비즈니스 로직과 분리
  + *경량 컨테이너* / 전체 WAS 없이도 단독 실행 가능한 구조
  + -> 복잡성 제거 및 기능성 유지의 절묘한 균형 제공
+ 프레임워크와 라이브러리의 차이점을 제어 흐름의 주체와 사용 방식을 중심으로 설명하고, Spring Framework와 일반 Java 라이브러리를 예시로 들어 설명하세요.
  + 프레임워크는 **제어 흐름의 주체가 프레임워크**이며 **라이브러리는 개발자**이다.
  + Framework는 종속되며(프레임워크에 맞춰 개발해야 함), Library는 작은 기능을 개발해 둔 것으로 가져다 사용하는게 자유롭다. (쓸 수도 안 쓸수도 있음)
  + Spring Framework의 경우 Bean을 이용해 객체를 생성, 관리, 소멸을 컨테이너가 대신 해준다. -> 대신 관련 설정을 수동으로 작성해야 했고, 사용하지 않는 기능에 대해서도 정의했어야 했다.
  + Java Library의 경우 import를 통해 해당 Library에서 제공하는 기능을 손쉽게 가져다 사용하는게 가능하다.
  + Library에서 제공하는 모든 기능을 사용(정의)하지 않아도 된다. 사용하고 싶은 것만 사용해도 된다.




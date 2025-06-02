# WEEKLY_PAPER
- [2주차](#2주차) [rebase와 merge / fetch와 pull]
- [3주차](#3주차) [단일책임원칙(SRP)와 개방-폐쇄원칙(OCP) / stream 연산자 map()과 flatMap()]
- [4주차](#4주차)
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
    + 기능별로 분리, 관심사의 분리, 하나의 책임만 갖도록 설계하여 적용할 수 있다.
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
  + **개방-폐쇄 원칙(OCP)는 기존 코드를 변경하지 않고, 새로운 기능은 확장을 통해 추가할 수 있도록 설계하는 설계 원칙이다.
    + 핵심 인터페이스 정의 -> 다양한 구현체 -> 런타임에 전략 주입은 OCP에 아주 적합한 구조이다.
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
  + map은 원하는 필드만 추출하거나 특정 형태로 변환할 때 사용되는 중간 연산자이다.
    + 스트림의 요소 변환은 원본 객체에 영향을 주지 않는다.
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
  + flatMap()은 중첩된 스트림 구조를 평탄화하여 하나의 스트림으로 펼쳐주는 연산자이다.
    + 이중 이상의 리스트의 경우 하나의 자료를 컨트롤하기 복잡하기 때문에 이 flatMap을 사용하여 평탄화를 한 뒤 평탄화를 진행하기도 한다.
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














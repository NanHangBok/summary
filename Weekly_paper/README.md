# WEEKLY_PAPER
- [2주차](#2주차) [rebase와 merge / fetch와 pull]
- [3주차](#3주차) [단일책임원칙(SRP)와 개방-폐쇄원칙(OCP) / stream 연산자 map()과 flatMap()]
- [4주차](#4주차) [해시셋 / O(n)과 O(log n)]
- [5주차](#5주차) [Spring Framework 등장 배경 / Framework vs Library]
- [6주차](#6주차) [웹 서버와 WAS (tomcat은 어디?) / Spring boot에 Bean등록 방법 및 장단점]
- [7주차](#7주차) [AOP / Controller와 RestController]
- [8주차](#8주차) [SOAP과 REST / @RestController 처리 과정]
- [9주차](#9주차)

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
  + 해결하고자 했던 문제점 remind
    + 기존 EJB의 복잡한 사용법을 간편하게(높은 기술 종속성) -> POJO 기반 프로그래밍로 단순한 Java객체만으로 비즈니스 로직 구현
    + 무거운 컨테이너 경량화 -> 경량 컨테이너(IoC 컨테이너, Tomcat없이 실행 가능)
    + 복잡한 설정 -> Spring Framework가 알아서 처리 (어노테이션 기반)
    + 테스트의 어려움 -> 단위 테스트의 용이, 테스트용 객체 주입(Mock)

+ 프레임워크와 라이브러리의 차이점을 제어 흐름의 주체와 사용 방식을 중심으로 설명하고, Spring Framework와 일반 Java 라이브러리를 예시로 들어 설명하세요.
  + 프레임워크는 **제어 흐름의 주체가 프레임워크**이며 **라이브러리는 개발자**이다.
  + Framework는 종속되며(프레임워크에 맞춰 개발해야 함), Library는 작은 기능을 개발해 둔 것으로 가져다 사용하는게 자유롭다. (쓸 수도 안 쓸수도 있음)
  + Spring Framework의 경우 Bean을 이용해 객체를 생성, 관리, 소멸을 컨테이너가 대신 해준다. -> 대신 관련 설정을 수동으로 작성해야 했고, 사용하지 않는 기능에 대해서도 정의했어야 했다.
    + 개발자는 Spring Framework에서 제공하는 어노테이션(@Controller, @Service 등)을 사용하여 프레임워크에게 전체 흐름 제어를 요청한다.
    + Spring이 실행 시점에 해당 어노테이션에 알맞은 메서드를 호출하며 IoC컨테이너가 모든 객체의 생성과 주입을 관리한다.
    + 이 과정에서 Spring 내부적으로 필요한 Bean 객체도 생성되는데 이는 개발자가 컨트롤하지 못하는 Spring Framework가 사용하는 객체이다.
  + Java Library의 경우 import를 통해 해당 Library에서 제공하는 기능을 손쉽게 가져다 사용하는게 가능하다.
    + Library에서 제공하는 모든 기능을 사용(정의)하지 않아도 된다. 사용하고 싶은 것만 사용해도 된다.
    + 개발자가 필요할 때 마다 직접 호출해서 사용하게 된다.
    + 흐름을 개발자가 제어하게 됨.

---

## 6주차
+ 웹 서버(Web Server)와 WAS(Web Application Server)의 차이를 설명하고, Spring Boot의 내장 톰캣이 이 둘 중 어디에 해당하는지 설명해주세요.
  + Web Server는 정적 데이터(이미지, 오디오, 비디오 등) 혹은 모든 사용자가 **동일한 화면**을 보는 **정적 HTML**을 반환하는 서버이다.
  + WAS는 웹 서버 이상의 일을 하며(웹 서버를 포함하며) 여러 개의 작업을 진행하며 **사용자마다** 화면이 다른 **동적 HTML**을 반환한다.
    + 클라이언트의 요청을 받아 비즈니스 로직을 실행하고, DB에 접근 하는 등 결과를 **웹 서버를 통해 전달**하게 된다.
    + Spring Boot의 **내장 톰캣이 여기에 해당**한다.

+ Spring Boot에서 사용되는 다양한 Bean 등록 방법들에 대해 설명하고, 각각의 장단점을 비교하세요.
  + 주로 *자바 기반 Bean 등록 방법*과 *컴포넌트 스캔을 통한 Bean 등록 방법*이 존재한다.
  + 자바 기반 Bean 등록 방법은 *@Configuration* 어노테이션을 사용하는 클래스에서 *@Bean* 어노테이션을 추가해서 Bean을 등록할 수 있다.
    + @Configuration 어노테이션에는 내부적으로 @Component가 포함된다. 컴포넌트 스캔의 대상이 된다.
    + @Bean 어노테이션을 *메서드 단위에 추가*하여 메서드의 **반환 객체를 Bean으로 등록**하게 된다.
    + @Bean 메서드 명이 Bean의 이름으로 등록된다. / 중복 될 수 는 없으며 동일 이름이 있을 경우 덮어쓰기 또는 예외가 발생한다.
    + @Bean(name="asdf")를 통해 명시적 이름을 부여할 수 있다.
      <br><br>
    + 직접 인스턴스 생성 로직 제어 가능 -> 인스턴스를 직접 생성하기 때문에 외부 설정 기반 조건 처리에 유연하게 대응가능
      + application.yaml 값 기반 조건 처리가 가능(쉬움)
    + 코드가 길어지고 복잡해 질 수 있음
    + 자동 등록보다 DI 구성 흐름이 명확하지 않을 수 있음
   
  + 컴포넌트 스캔 방식은 @Component 어노테이션을 통해 특정 패키지 이하의 클래스들을 **자동으로** 탐색하여 Bean으로 등록할 수 있다.
    + *기본 탐색 시작점*은 **해당 어노테이션이 정의된 클래스가 위치한 패키지**이다.
    + @Component, @Controller, @Service, @Repository, @Configuration 등을 통해 등록한다.
    + basePackages 옵션을 사용하여 탐색 경로를 수동으로 지정할 수 있으며, 다수의 패키지 경로를 지정할 수 있다.
    + basePackagesClasses 옵션을 통해 클래스 기반으로 탐색 범위를 정할 수 있다.
    + includeFilters, excludeFilters 옵션을 통해 포함/제외 대상을 필터링할 수 있다.
     <br><br>
    + 어노테이션만 붙이면 되기 때문에 간단하고 직관적
    + 자동 스캔이 되므로 빠르게 개발을 할 수 있음
    + 등록 조건을 유연하게 제어하기 어려움
   
  + @SpringBootConfiguration, @EnableAutoConfiguration 등을 통해 *스프링 내부에서 사용하는 Bean이 자동으로 등록되기도 함*
 
  + 그 외에도 xml을 통한 bean 생성 방식이 있긴 한데 최근에는 잘 사용되지 않는 레거시 방식이다.

---
## 7주차

+ Spring에서 AOP(Aspect Oriented Programming)가 필요한 이유와 이를 활용한 실제 애플리케이션 개발 사례에 대해 설명하세요.
  + **핵심 비즈니스 로직과는 별개인 공통 기능(횡단 관심사)을 하나의 모듈(Aspect)로 분리하여 관리** (관심사의 분리)
  + 로깅, 트랙잭션 처리, 보안 검사, 실행시간 측정 등 여러 클래스에 분산되어 반복되기 쉬운 로직을 Aspect라고 부른다.
    + 이러한 공통 기능을 분리하여 관리하기 위해 ( 중복 제거 및 코드 유지보수성을 위해 )
    + @Before, @After @Around 등 어노테이션을 통해 시점을 결정한다.
    + @Transactional을 통해 트랜잭션 처리를 하기도 한다.
  + Spring은 @AspectJ 스타일의 AOJ를 지원하며 AspectJ 전체 기능 중 프록시 기반 AOP를 기본 제공 한다.

+ Spring MVC에서 클라이언트의 요청 처리 흐름을 @Controller와 @RestController의 차이점을 중심으로 각각의 처리 과정과 특징을 포함하여 설명하세오.
  + @Controller
    + View 기반 MVC에서 사용.
    + 주로 Thymeleaf와 연동
    + defualt
    + View 이름 반환

  + @RestController
    + REST API 응답을 위한 컨트롤러
    + 자동으로 @ResponseBody 포함 ( @Controller + @ResponseBody )
    + 주로 JSON 반환 시 사용.
    + 객체 또는 문자열 반환

  + 클라이언트로부터 요청이 들어오면 HandlerMapping과 HandlerAdapter를 통해 적절한 컨트롤러가 Handler 메서드를 호출하게 되는데 이 때 @Controller의 경우는 문자열이 들어 있을 때, **ViewResolver + View**를 통해 해당 문자열과 매핑이 되는 View를 반환하게 된다. 문자열을 반환하고 싶을 때는 @ResponseBody를 추가해야 한다. 반면에 @RestController의 경우 문자열을 반환하게 될 시 **HttpMessageConverter**를 통해 반환 **객체를 자동으로 JSON으로 변환**하여 HTTP Body에 담아 응답하게 된다.
  + 웹 애플리케이션(@Controller) / 렌더링 처리를 ViewResolver + View가 담당 / 결과가 View
  + REST API 서버 (@RestController) / 렌더링 처리를 HttpMessageConverter가 담당 / 결과가 JSON 데이터
  + 사용대상이 View는 브라우저 기반 SSR 앱 / JSON 응답은 SPA, 모바일 앱, API 통신

---
## 8주차

+ 웹 API의 발전 과정에서 SOAP에서 REST로의 전환이 일어난 이유와 그 장단점에 대해 설명하세요.
  + soap의 단점 및 한계가 있었다
    + XML 구조가 지나치게 무겁고 장황함 -> 네트워크 트래픽 증가, 파싱 성능 저하
    + 메시지 해석과 작성에 높은 복잡도 -> 개발자가 직접 다루기 힘듦
    + WSDL 학습 비용(러닝커브)가 큼. 자동 코드 생성 도구가 없으면 개발 속도 저하
    + 브라우저 기반 호출이나 REST 클라이언트와 호환성 떨어짐
  + REST의 경우 구조가 단순하고 HTTP 메서드와 자원 중심 URI 구조로 일관되게 작성되서 처음 보는 API도 예측이 쉬워진다
  + 규칙만 잘 지키면 쉽게 확장할 수 있고 URI 설계 원칙을 유지하면서 자연스럽게 확장이 가능하다.

  + SOAP 장단점
    + 계약 기반 -> WSDL을 통해 서비스 정의 -> 클라이언트가 자동으로 인터페이스 생성
    + 엄격한 스키마 검증 -> XSD 기반 스키마를 따름. 파라미터나 리턴 값의 구조가 명확하게 정의됨
    + HTTP 외에도 SMTP, FTP 등 다양한 프로토콜에서 동작 가능
    + 보안 및 신뢰성 확장 기능 제공
    + SOAP 메시지 내부에서 ACID 트랜잭션 관리 가능

  + REST API 장단점
    + UI가 바로 변경되는 것을 막고, 규칙적 분리
    + Stateless. 각 호출은 반복되는 정보와 문맥을 가지고 있음
    + 코드 재사용성이 늘어난다.
    + *구현 내용*을 은닉할 수 있다.
    + **표준화된 데이터 교환**이 가능하다.
    + **일관성과 확장성**이 좋다.
      + 일관성 : HTTP 메서드와 자원 중심 URI 구조를 일관되게 사용
        + **처음보는 API도 예측이 쉬워짐**
      + 확장성 : 자원 중심 설계라 규칙만 잘 지키면 쉽게 확장할 수 있음

+ Spring Boot에서 @RestController로 들어온 HTTP 요청이 처리되어 응답으로 변환되는 전체 과정을 설명하세요. 특히 HTTP 메시지 컨버터가 동작하는 시점과 역할을 포함해서 설명하세요.
  + HTTP 요청이 들어오면 (EntryPoint)**DispatcherServlet**이 매핑되는 핸들러메서드를 HandlerMapping을 통해 찾은 뒤 HandlerAdapter로 결과를 위임해서 HandlerAdapter는 해당하는 컨트롤러를 찾아 메서드를 실행해 결과를 얻는다. 이 때 해당 컨트롤러가 @RestController 라면 결과가 View이름을 반환하지 않고 **Text나 객체를 반환**하게 되고 **반환되는 결과값은 HttpMessageConverter로** 넘겨져서 해당 클래스(MessageConverter)는 Jackson 라이브러리를 통해 **JSON으로 반환**되어 DispatcherServlet으로 다시 반환되어 해당 **JSON 결과를 클라이언트(프론트엔드)로** 넘겨준다.

  + 컨트롤러에서 객체를 반환할 때 **ResponseEntity**를 반환하게 되면 ***HTTP 상태 코드나 헤더 내용을 추가하거나 수정***해서 클라이언트(프론트엔드)로 넘겨 줄 수 있다.

  + 응답은 보통 **일관된 결과**를 얻기 위해 data 필드 안에 JSON 결과를 넣는다. error는 비워둔다. 오류가 있을 시에는 data를 비워두고 error에 error내역을 담는다.

  + MessageConverter는 총 2번 동작한다.
    + 요청 처리 시, HandlerAdapter가 컨트롤러의 파라미터를 분석할 때 ArgumentResolver가 동작한다.
        + 이때 @RequestBody가 붙은 **객체 파라미터**는, **요청 본문의 JSON 데이터를 MessageConverter가 해당 객체의 필드와 1:1로 매핑하여 Java 객체로 변환**한다.
    + 컨트롤러는 이 변환된 객체를 사용해 로직을 수행하고, 리턴 객체를 반환한다.
    + **응답 처리 시**, HandlerAdapter는 이 리턴 객체를 다시 **MessageConverter를 통해 JSON 문자열로 직렬화**한다.
    + 직렬화된 JSON은 DispatcherServlet을 통해 **클라이언트에게 HTTP 응답 본문으로 전송**된다.
    + *만약 객체 파라미터를 사용하지 않는다면 MessageConverter는 동작하지 않는다*.

---

## 9주차

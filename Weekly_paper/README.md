# WEEKLY_PAPER
- [2주차](#2주차) [rebase와 merge / fetch와 pull]
- [3주차](#3주차) [단일책임원칙(SRP)와 개방-폐쇄원칙(OCP) / stream 연산자 map()과 flatMap()]
- [4주차](#4주차) [해시셋 / O(n)과 O(log n)]
- [5주차](#5주차) [Spring Framework 등장 배경 / Framework vs Library]
- [6주차](#6주차) [웹 서버와 WAS (tomcat은 어디?) / Spring boot에 Bean등록 방법 및 장단점]
- [7주차](#7주차) [AOP / Controller와 RestController]
- [8주차](#8주차) [SOAP과 REST / @RestController 처리 과정]
- [9주차](#9주차) [DDL과 DML / 역정규화]
- [10주차](#10주차) [N+1 / ACID 중 Isolation, 격리수준]
- [13주차](#13주차) [각 계층에서 수행되는 입력값 검증 / Mockito ]
- [14주차](#14주차) [컨테이너 기술과 Docker / 컨테이너 오케스트레이션]
- [15주차](#15주차) [RDS / GitHub Actions]
- [19주차](#19주차) [세션 기반과 토큰기반 / OAuth 2.0]
- [20주차](#20주차) [웹 애플리케이션 4가지 주요 보안 공격 / JWT]
- [21주차](#21주차) [경쟁 상태 / 비동기 환경 컨텍스트 정보 전달]
- [22주차](#22주차) [Spring Cache / 로컬 캐시와 분산캐시]
- [23주차](#23주차) [OSI 7계층과 TCP/IP 4계층 / TCP와 UDP]

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

+ SQL에서 DDL과 DML의 차이점을 설명하고, 각각의 대표적인 명령어들의 용도를 설명하세요.
  + DDL : Data Definition Language (데이터 정의어) : 테이블과 같은 데이터 구조를 정의하는데 사용되는 명령어들 ( 생성, 변경, 삭제, 이름변경 등) 데이터 구조와 관련된 명령어들을 말함
    + Create : 테이블 생성
      ```sql
      CREATE TABLE [table_name](
        [column1] uuid PRIMARY KEY,
        [column2] varchar(100) [UNIQUE | NOT NULL ...],
        [column3] uuid,
        FOREIGN KEY ([fk_column]) REFERENCES [fk_table](fk_column) ON DELETE CASCADE
      );
      ```
    + Alter : 테이블 구조 수정
    + Rename : 테이블 이름 수정
    + Drop : 테이블 삭제
    + Truncate : 테이블 삭제
    + Drop vs Truncate
      + Drop 은 table 자체를 삭제한다. 해당 테이블은 처음부터 없었던 테이블처럼 사라지며, rollback이 불가능 하다.
      + Truncate는 테이블 내용을 삭제하는 것으로, 테이블의 구조는 유지한 채 최초 생성 시 처럼 데이터가 들어있지 않은 빈 테이블 상태로 만드는 명령어다. 이 또한 rollback이 불가능하다.

  + DML : Data Manipulation Language (데이터 조작어) : 데이터 베이스에 들어 있는 데이터를 조회하거나 검색, 수정 및 삭제 등 명령어를 말하는 것
    + Select : 데이터 조회
      + ```sql
        SELECT * FROM [Table]
        WHERE [조건] -- 그룹핑 이전 필터링 / GROUP BY 없이 사용 가능
        GROUP BY [그룹핑할 컬럼(어트리뷰트)] -- 그룹핑을 통해 집계 함수 사용 가능
        [HAVING [조건]] -- 그룹핑 이후 필터링 / 집계함수 사용 가능
        ORDER BY [정렬 기준 컬럼] [ASC|DESC] -- Default는 ASC 오름차순 / 여러 개 컬럼으로 정렬 가능, 이때 제일 왼쪽 컬럼이 첫번째 정렬 기준이 된다.
        ```
        ```sql
        -- JOIN
        SELECT * FROM [Table] as t -- t는 별칭 as 생략 가능
        [INNER|LEFT OUTER|RIGHT OUTER] JOIN [Another Table] as a -- JOIN 앞 생략 시 INNER JOIN / OUTER 생략 가능
        ON t.id = a.t_id -- 보통 참조하는 테이블의 FK와 참조되는 테이블의 PK / 단순히 두 테이블 간의 연결고리를 하나로 합쳐준다고 보면 된다.
        -- JOIN 여러 번 가능
        ```
    + Insert : 데이터 추가
      + ```sql
        INSERT INTO [Table] ([column1, column2...])
        VALUES ([data1,data2...]) [, ([data1,data2])...] -- 여러개 동시 삽입 가능
        -- 위에서 지정한 컬럼의 순서와 VALUES의 입력값의 순서가 일치하여야 한다.
        ```
    + Update : 데이터 수정
      + ```sql
        UPDATE [Table]
        SET [column] = [data] -- 수정할 컬럼명과 수정할 값
        WHERE [column(PK,UNIQUE 혹은 기준컬럼명)] = [data]; -- 조건에 맞는 행만 수정 (ex: id가 3, 나이가 20살 이상 etc...)
        -- WHERE 조건이 없으면 모든 행이 수정된다.
        ```
    + Delete : 데이터 삭제
      + ```sql
        DELETE FROM [Table]
        WHERE [조건];
        -- UPDATE와 마찬가지로 WHERE이 테이블 내 모든 행이 사라진다.
        ```
   
+ 역정규화가 필요한 상황과 적용 시 고려해야 할 사항, 그리고 역정규화를 적용할 때의 장단점을 설명해주세요.
  + 아래와 같은 상황에 성능 또는 관리 편의성 등을 이유로 일부러 정규화를 완화하는 설계 기법
    + 테이블 조인 횟수가 많고, 성능 저하가 발생하는 경우
    + 자주 사용되는 복잡한 쿼리가 반복되는 경우 -> 집계나 통계가 반복적으로 필요한 경우
    + 데이터의 실시간성보다 빠른 조회가 중요한 경우

 
  + 고려해야 할 사항
    + 성능 병목 구간을 명확히 파악한 후 적용해야 한다.
    + 정규화를 낮추는 방법이기 때문에 **데이터 중복으로 정합성(일관성) 문제가 발생할 수 있고**, 유지보수가 까다로워 질 수 있기 때문에
      + **인덱스, 캐시, 뷰 등 다른 방법으로 해결 가능한지 먼저 고려해야 한다.**
    + **일관성을 유지하기 위한 추가 로직이 필요**하고, / 배치 처리, 트리거, 어플리케이션에서 갱신 책임을 분리할 필요가 있다.
   
  + 장점
    + JOIN이 감소하고, 그로 인해 조회 속도가 향상된다.
    + 데이터 구조가 단순해지고, 특정 쿼리에서 편리하게 조회할 수 있다.
    + 통계, 보고서 등 **JOIN이 많이 들어가고 읽기 중심 작업**(반복 조회)에 적용 시 성능이 유리해 진다.
   
  + 단점
    + 일부 데이터의 중복을 허용해 정합성 문제가 발생한다.
    + 갱신 복잡성이 증가한다.
      + INSERT, UPDATE, DELETE 시 일관성을 유지하기 위한 **추가적인 갱신 로직이 필요하다** / 데이터 불일치를 막기 위해
      + 스키마 변경 시 영향 범위가 넓어진다.
        + 중복된 컬럼이 여러 테이블에 존재한다면 해당 컬럼의 삭제 및 변경 시 여러 테이블을 동시에 변경할 필요가 있다.
        + 컬럼이 추가 될 시 관련 테이블에 추가적으로 고려해야 한다.
    + 읽기 성능은 좋아 질 수 있으나 유지보수에 비용이 추가적으로 들어갈 수 있다.
   
  + 역정규화 이후 인덱스를 추가하는 등 튜닝을 병행하여 조회 속도를 더욱 빠르게 할 수 있다.
    + --> 빠른 조회를 위해 역정규화를 하기 때문에 중복 컬럼을 생성하고 인덱스를 추가하여 조회 성능을 향상시킨다.

---

## 10주차

+ JPA에서 발생하는 N+1 문제의 발생 원인과 해결 방안에 대해 설명하세요.
  + N+1 문제란?
    > ORM(JPA, Hiberate) 환경에서 1번의 쿼리로 N개의 데이터를 조회한 후, 각 데이터의 연관된 데이터를 추가 조회하면서 총 N+1번의 쿼리가 발생하는 문제
    > > 회원을 10명 조회했는데 각 회원의 주문 정보를 또 조회하면, 회원 1명 당 1번의 쿼리, 총 10번 발생 > 1 + N(10) = 11번의 쿼리
  + 발생 이유
    + 지연 로딩(Lazy) 전략으로 연관 관계가 설정된 엔티티를 조회할 때, 각각 개별적으로 추가 조회가 발생하기 때문
    + ORM에서 테이블-엔티티 맵핑 시 JOIN을 사용하면 1번의 쿼리로 결과를 얻을 수 있는 상황에 엔티티마다 쿼리문을 사용하여 조회하기 때문에 추가 쿼리가 발생하는 상황
    + **참조하는 테이블에서 참조된(원본) 테이블의 정보를 얻기 위해 추가 쿼리가 발생하는 상황**
    + > LAZY는 실제 객체를 사용할 때마다 DB에 접근<br>
      > 객체로 매핑 시 참조되는 테이블의 값 까지는 알 수 없음.<br>
      > 참조되는 테이블의 값이 필요할 때 마다 쿼리를 추가 실행 -> N+1번 문제 발생
   
  + 문제점
    + DB 쿼리가 불필요하게 N번 추가 실행되기 때문에 **성능 저하**를 일으킨다.
    + 트랜잭션이 불필요하게 길어지기 때문에 **복잡도가 증가**한다.
    + 숨어 있는 쿼리가 발생할 수 있기 때문에 **예측이 어려워진다**.
   
  + 해결 방안
    + Fetch Join
      + JPQL에서 JOIN FETCH 구문을 사용하면, 연관된 엔티티를 한 번에 조회.
      + OneToMany Fetch Joint 시 중복 발생 -> 컬렉션에서는 DISTINCT 사용 권장
      + 페이징 불가 (데이터가 부풀려짐)
      + **페이징 불가, 중복 데이터 가능**
     
    + EntityGraph
      + 쿼리를 작성하지 않고 fetch 전략을 재정의할 수 있는 방법
      + 특정 쿼리에만 Eager처럼 작동하게 변경 가능. JPQL 없이 선언적으로 연관 데이터 조회 가능.
      + ```
        @EntityGraph(attributePaths = {"orders"})
        ```
      + 다양한 쿼리에서 재사용 가능, 선언적 방식
      + 페이징 시에도 사용 가능.

+ 트랜잭션의 ACID 속성 중 격리성(Isolation)이 보장되지 않을 때 발생할 수 있는 문제점들을 설명하고, 이를 해결하기 위한 트랜잭션 격리 수준들을 설명하세요.
  + 격리성(Isolation) ?
    + 여러 트랜잭션이 동시에 수행될 때, 각 트랜잭션이 서로 영향을 받지 않도록 분리되어야 한다.
    + ex : 두 사용자가 동시에 같은 상품을 구매할 때, 재고 차감이 꼬이지 않도록 트랜잭션 격리 필요.
  + 동시성 제어의 핵심
    + 적절한 격리 수준 설정을 통해 데이터 무결성과 성능 사이의 균형을 맞출 수 있다.
    + **낮은 격리 수준**은 높은 동시성과 성능을 제공하지만 **데이터 일관성 문제**를 발생 시킬 수 있음.
    + 높은 격리 수준은 데이터 일관성을 보장하지만 동시성과 성능이 저하될 수 있음.
   
  + 문제점
    + Dirty Read
      + 커밋되지 않은 데이터를 다른 트랜잭션이 읽은 현상
      + 확정되지 않은 값으로 잘못된 로직이 수행될 수 있어 금융, 결제 시스템에서는 절대 허용되지 않음.
      > ex : A 트랜잭션이 계좌 잔액을 수정함.<br>
      커밋되지 않은 상태에서 B 트랜잭션이 해당 데이터를 읽음.<br>
      이후 A 트랜잭션이 롤백될 경우 B는 잘못된 데이터를 읽은 것

    + Non-repeatable Read
      + 같은 쿼리를 실행했을 때 이전과 다른 결과가 나오는 현상
      > ex : A 트랜잭션이 상품 가격을 읽었는데 <br>
      > B 트랜잭션이 해당 가격을 수정하고 커밋한 후 A가 다시 읽으면 다른 가격이 조회 됨.
  
    + Phantom Read
      + 일정 범위의 레코드를 두 번 이상 읽을 때 이전에 없던 레코드가 나타나는 현상
      > ex : A 트랜잭션이 '회원 수 >= 30' 조건으로 조회했는데 <br>
      > B 트랜잭션이 새 회원을 추가하고 커밋하면, <br>
      > A가 다시 조회할 때 회원 수가 달라짐.

  + 격리 수준
    + **@Transactional(isolation = Isolation.~~) 로 설정 가능**
    + READ UNCOMMITTED
      + 가장 낮은 격리 수준
      + 커밋되지 않은 데이터를 읽을 수 있음.
      + Dirty Read, Non-repeatable Read, Phantom Read 모두 발생
      + 실시간 모니터링 등 정확성보다 속도가 중요한 경우 사용
    + READ COMMITTED
      + Dirty Read는 방지되지만 Non-repeatable Read, Phantom Read는 발생 가능
      + 일반적인 서비스에서 많이 사용, 적절한 일관성과 성능 제공.
    + REPEATABLE READ
      + 트랜잭션이 시작될 때 스냅샷을 생성하고, 끝날 때까지 동일한 데이터만 조회함
      + Dirty Read, Non-repeatable Read 방지, Phantom Read는 발생 가능
      + 트랜잭션 동안 동일 데이터를 반복 조회 시 값이 유지됨
    + SERIALIZABLE
      + 가장 높은 격리 수준, 모든 트랜잭션을 순차 실행처럼 보이게 함
      + Dirty Read, Non-repeatable Read, Phantom Read 모두 방지
      + 성능 저하 및 교착 상태 발생 가능, 높은 일관성 필요 시 사용
    + 교착상태(Deadlock)
      + 두 개 이상의 프로세스(또는 트랜잭션)가 서로 상대방의 자우너을 기다리며 무한 대기하는 상태
      > A는 B의 자원을 기다리고<br>
      > B는 A의 자원을 기다리는 상황 => **서로 기다리느라 끝나지 않음**

---

## 13주차
+ 애플리케이션의 각 계층에서 수행되는 입력값 검증의 범위와 책임을 어떻게 나눌 것인지에 대해 설명해주세요. 특히 중복 검증을 피하면서도 안정성을 확보하는 방안과, 이와 관련된 트레이드오프에 대해 설명해주세요.
  + Controller : 보통 웹에서 들어오는 값의 정합성을 검증한다.
    + +DTO를 통한 정합성 검증
    + @Valid 또는 @Validated를 적용하여 요청 데이터를 자동으로 검증 가능.
    + 검증 실패 시 BindingResult를 통해 오류 정보를 확인하고 적절한 응답을 반환 가능.
    + ```
      @PostMapping("/users")
      public String createUser(@Valid UserForm userForm, BindingResult bindingResult, Model model) {
          if (bindingResult.hasErrors()) {
              return "users/form";  // 검증 실패 시 폼으로 다시 이동
          }
          userService.create(userForm);
          return "redirect:/users";  // 검증 성공 시 리다이렉트
      }
      ```
    + @Valid 또는 @Validated가 적용된 파라미터 바로 다음에 위치해야 함.
    + @Valid = Bean 검증
    + 메서드 파라미터 검증 = 클래스에 @Validated 사용, @NotNull등 어노테이션 사용
      + ```
        public class MemberRequest {
            @NotNull(groups = Update.class) // 수정 시만 필수
            private Long id;
        
            @NotBlank(groups = {Create.class, Update.class}) // 생성·수정 모두 필수
            private String name;
        
            interface Create {}
            interface Update {}
        }
        
        @PostMapping("/members")
        public void create(@Validated(MemberRequest.Create.class) @RequestBody MemberRequest req) {
            // Create 그룹에 해당하는 필드만 검증
        }
        
        @PutMapping("/members")
        public void update(@Validated(MemberRequest.Update.class) @RequestBody MemberRequest req) {
            // Update 그룹에 해당하는 필드만 검증
        }
        ```
  + Service : 컨트롤러보다 더 복잡한 비즈니스 규칙을 검증하거나 도메인 객체 간의 관계 검증 등을 수행할 수 있다.
    + 복잡한 비즈니스 규칙 ex) 출발 계좌와 목적 계좌는 같을 수 없다.
    + 복잡한 비즈니스 검증을 재사용 가능하다.
    + 도메인 중심의 책임 분리가 가능하다.
  + Repository : DB 제약을 통해 데이터 정합성을 검증한다.
 
  + 중복검증 : 데이터의 정합성을 웹, Controller, Service, Repository 모든 계층에서 검증 시 제약 조건이 변경되면 모든 코드를 바꿔야 한다.
  + 입력값 검증의 범위와 책임을 명확히 나누면 중복 검증을 피할 수 있고 코드의 유지보수가 쉬워진다.
  + Controller에서 복잡한 비즈니스 로직까지 검증하면 Service 계층과 코드가 중복되는 경우가 생길 수 있고
  + Service에서 모든 검증을 처리하게 되면 검증되지 않은 데이터로 불필요한 호출이 발생할 수 있다.
  + 위에서 언급한 groups를 통한 그룹 검증을 무분별하게 사용 시 관리 복잡도가 증가한다.
  + Repository까지 검증 필요성이 내려오면 DB호출이 증가하게 되고 결국 예외 처리는 서비스에서 하게되는 불필요한 호출이 증가한다.

+ 테스트에서 사용되는 Mockito의 Mock, Stub, Spy 개념을 각각 설명하고, 어떤 상황에서 어떤 방식을 선택해야 하는지 구체적인 예시와 함께 설명하세요.
  + Stub : 고정된 응답 반환, 테스트 대상 외부 동작을 하드 코딩하여 제공, 간단한 조건/결과가 필요한 테스트
  + Mock : 호출 여부/순서/횟수 검증/ 행동 설정 가능, 미리 동작 정의하거나 호출 결과를 검증, 외부 시스템이 복잡하거나 테스트 검증이 필요한 경우
  + Spy : 실제 객체를 감싸서 일부 메서드만 mocking, 실제 객체와 결합되어 실제 동작도 가능, 일부만 대체하고 나머지는 실제 동작해야 할 때
  + ```java
    // Stub 예시
    EmailService stubEmailService = new EmailService() {
        @Override
        public boolean sendEmail(String to, String message) {
            return true; // 항상 성공
        }
    };
    // 고정된 응답을 반환하기 때문에 단순한 결과(boolean 상태값, 단순한 리턴값)를 확인하고 싶을 때 사용
    ```
    
  + ```java
    // Mock 예시 (Mockito 사용)
    EmailService mockEmailService = mock(EmailService.class);
    when(mockEmailService.sendEmail(anyString(), anyString()))
    	.thenReturn(true);
    when(mockEmailService.sendEmail("김러키", "귀여워"))
    	.thenReturn(true);
    verify(mockEmailService, times(1)).sendEmail("user@example.com", "가입을 환영합니다");
    // 메서드의 인자값, 호출 여부, 호출 횟수 등을 검증하고 싶을 때 사용
    // 단순히 결과를 확인하기보다는 호출(행위)를 검증하고 싶을 때
    // 외부 시스템이 복잡하거나 테스트 검증이 필요한 경우
    ```
    
  + ```java
    // Spy 예시 (일부만 mocking)
    EmailService realService = new EmailService();
    EmailService spyService = spy(realService);
    doReturn(true).when(spyService).sendEmail(anyString(), anyString());
    // 실제 객체를 spy객체로 감싸서 실행.
    // 일부 메서드만 mocking
    // doReturn(true).when(spyService).sendEmail(anyString(), anyString());는 실행이 아닌 설정 코드
    // 일부만 대체하고 나머지는 실제 동작해야 할 때 사용
    // 검증 로직을 무시한채 비즈니스 로직이 실행하는 지 확인할 때?
    ```

---

## 14주차
- 컨테이너 기술과 Docker를 명확히 구분하여 설명하세요. 컨테이너 기술이 Docker 이전에도 존재했던 개념임을 언급하고, Docker가 컨테이너 기술을 구현한 하나의 도구라는 관점에서 설명해주세요. 또한, Docker 외에 컨테이너 기술을 구현한 다른 도구의 예시를 들어보세요.
  - **컨테이너**
    - 운영체제 수준에서 격리된 실행 환경.
    - 컨테이너 내부에서 실행되는 애플리케이션은 마치 독립된 OS 위에서 실행되는 것처럼 작동하지만, 실제로는 호스트 OS의 커널을 공유
      - 커널
        ```
        운영체제(OS)의 핵심이자 최하위 계층에서 동작하는 소프트웨어
        하드웨어와 응용 프로그램 사이의 Bridge(중간다리) 역할을 수행.
        - 프로세스를 나눠주고 제어, 메모리 제어, 프로그램이 운영체제에 요구하는 시스템 콜 등을 수행
        ```
    - 가상머신보다 가볍고 빠르게 실행 가능
  - **Docker**
    - 컨테이너 기술을 손쉽게 사용할 수 있도록 만들어진 오픈소스 플랫폼
    - 도커 컨테이너는 도커 이미지로부터 생성됨.
      - Docker Image
        ```
        컨테이너를 실행하기 위한 정적인 실행패키지
        운영체제의 최소 구성요소, 애플리케이션 실행파일, 의존 라이브러리, 설정 파일, 환경 변수, Dockerfile에 정의된 명령들 이 포함됨
        특징 :
          - 불변 : 한 번 생성되면 수정 불가
          - 레이어 구조 : 효율적인 저장 및 캐시 활용 가능
          - 재사용 가능 : 동일 이미지로 수백 개의 컨테이너 생성 가능
        ```
    - 시스템 전체가 아니라 애플리케이션 단위로 패키징되어 있음
    - 하나의 이미지만 있으면 Windows, macOS, Linux, 클라우드 어디서든 동일하게 실행됨
      - 개발 환경의 차이에서 발생하는 오류를 최소화하는 도구로 Docker가 널리 사용 됨.
  - **컨테이너와 Docker**
    - 컨테이너의 특징과 Docker
      - 하나의 컨테이너는 하나의 프로세스를 실행하는 데 최적화됨.
        > 시스템 전체가 아니라 애플리케이션 단위로 패키징 되어 있음
      - 다른 컨테이너와 파일 시스템, 네트워크, 환경 변수를 격리해서 사용함.
        > 독립적인 환경을 제공하며, 다른 컨테이너나 호스트와 격리됨
      - 가볍고 빠르게 실행됨
        > 실행이 매우 빠르고 자원 사용이 적음
    - ***위와 같은 방식으로 Docker는 컨테이너 방식을 구현하고 있는 하나의 도구이다.***
   
  - Docker = 빌드 + 실행 도구
  - *그 외 컨테이너 도구*
    - Podman
      - docker는 linux 컨테이너를 만들고 사용할 수 있도록 지원하는 컨테이너화 기술
        - Podman은 데몬이 없는 아키텍처를 보유.
        - 루트리스
      - docker는 컨테이너 생성 및 관리를 위한 올인원 툴
        - podman은 buildah 및 skopeo 등의 관련 툴은 컨테이너화의 특정 측면에 더욱 전문화
        - 사용자는 필요한 툴만 가지고 환경을 사용자 정의할 수 있음.
      - 사용자는 Podman에 대해 Docker의 별칭을 설정하거나 그 반대로 설정하여 간편하게 Podman과 Docker를 전환하여 사용 가능.
      - 쿠버네티스를 사용하지 않고 컨테이너를 실행하는 개발자에게 최적화
        - 쿠버네티스 컨테이너 오케스트레이션의 경우 CRI-O 사용 가능.
  - *컨테이너 빌드 도구*
    - Kubernetes는 docker에 대한 지원을 중단
    - Kaniko, Buildah, Buildkit
      - 세 도구 모두 OCI(Open Container Initiative) 규격을 따르는 컨테이너 이미지를 빌드 할 수 있음.
      - 세 도구 모두 Docker Daemon이나 유사한 Deamon을 필요로 하지 않아 이에 따른 자원의 소모량, 보안 문제를 해결 할 수 있음.
      - 세 도구들은 모두 이전 스테이지에서 빌드한 결과물을 다음 스테이지에서 사용할 수 있게 해주는 Multi-stage build 기능 가능.
        - Dockerfile을 간결하게 유지할 수 있으며 빌드의 효율성을 높일 수 있음.
    - Kaniko
      - Kaniko는 컨테이너 내부에서는 Root 권한을 가질 수 있지만 이 권한이 Host에까지 미치지 못함.
      - Snapshot 기능 존재.
        - Filesystem을 tarball로 압축해 Layer에 보관하는 캐싱 방법
        - 캐시를 외부 레포지토리에 저장하고 가져올 수 있는 Remote 캐싱 지원. CI/CD Pipeline 환경에서 사용하기 유리함.
      - Linux만 지원, Multi-architecture를 지원해 arm64, amd64 기반 이미지 빌드 가능
      - GKE Workload Identity 지원 : GKE 환경에서 이미지를 빌드할 시 Service account Key 없이도 권한을 부여할 수 있어 보안적인 장점 존재.
      - Profiling 지원 : 빌드 시간이 느릴 경우 Stacklog를 검사할 수 있는 Profiling 기능을 지원.
        - 이를 통해 빌드 시 문제점을 점검할 수 있음.
       
    - Buildkit :
      - 루트리스(rootless) 버전을 제공해 Root 권한 없이 이미지를 빌드할 수 있음.
        - seccomp와 AppArmor를 disable해야 하기 때문에 Linux Kernal 단에서 보안 취약점을 노출하게 된다는 단점 존재.
      - GCP의 GKE의 COS(Container Optimized OS)를 사용할 시 Rootless 모드에서 volume을 마운트할 수 없다는 이슈가 존재.
        - COS 환경에서는 Root 모드로 실행해야 한다는 제한이 있음.
      - Layer 단위로 캐싱을 수행해 이미지 빌드 시 각 Layer의 해시 값 비교를 통해 캐시 적중 여부를 판단.
      - 외부 레포지토리에 캐시를 저장하고 가져오는 Remote 캐싱 지원.
      - 이미지 자체에 캐시를 저장하는 Inline 캐싱 방식을 지원해 레포지토리를 간결하게 유지할 수 있음.
      - Linux, MacOS, Windows OS를 모두 지원.
      - 개발자의 로컬 빌드 테스트에 유리.
      - Multi-architecture 지원. arm64, amd64 기반 이미지를 빌드 할 수 있음.
      - Parallel Build 지원
        - Dockerfile에서 의존성이 없는 두 스테이지를 병렬로 실행해 빌드 속도를 높일 수 있음.
      - Opentracing 지원
        - Opentracing을 지원해 Jaeger와 같은 도구에서 Trace를 통합해 추적 가능.
    - Buildah
      - Rootless 버전 지원. Root 권한 없이 이미지를 빌드할 수 있음.
      - Dockerfile 기반 이미지 빌드는 Remote 캐싱 기능을 제공하지 않음.
      - Remote 캐싱은 스크립트 언어 기반으로 레이러를 추가하는 buildah add 명령어로만 가능.
      - Local Cache를 사용할 수 없는 CI/CD Pipeline 환경에서는 Docker 파일을 이용한 빌드 시 캐싱 기능을 사용할 수 없음.
      - Linux Os만 지원.
      - Mulit-architecture를 지원해 arm64, amd64 기반 이미지를 빌드할 수 있음.
      - Bash Script 지원
        - Dockerfile 대신 친숙한 Bash script를 사용하여 이미지 빌드 가능.
  - 참고 :
    - https://www.redhat.com/ko/topics/containers/what-is-podman
    - https://nangman14.tistory.com/92
- 컨테이너 오케스트레이션의 개념과 필요성을 설명하고, Docker 단독 사용 환경과 비교하여 컨테이너 오케스트레이션이 해결하는 주요 문제점 3가지(자동 확장, 자가 복구, 선언적 인프라)를 설명하세요.
  - 대규모 애플리케이션을 배포할 수 있도록 컨테이너의 네트워킹 및 관리를 자동화 하는 프로세스.
  - 관리형 컨테이너 오케스트레이션 플랫폼이 존재하기 전에는 조직에서 복잡한 스크립팅을 사용하여 여러 시스템에서 컨테이너 배포, 스케줄링 및 삭제를 관리.
    - 이러한 스크립트를 유지 관리하려면 버전 제어와 같은 문제가 발생. 설정을 확장하기도 어려움.
    - 이러한 복잡성을 자동화하고 해결하여 수동 관리와 관련된 문제를 제거.
  - Docker 단독 사용 :
    - 지속적인 모니터링 과 수동 개입 필요
      - 컨테이너 인스턴스를 수동으로 추가해야 함.
      - 트래픽이 줄어들 경우 불필요한 인스턴스가 생겨 리소스 낭비가 심해짐.
    - 서비스 중단 발생 및 장애 대응이 느려질 수 있음.
      - 서비스 장애가 발생하여 컨테이너가 비정상적으로 종료되면 해당 컨테이너에서 제공하던 서비스 중단.
    - 전체 시스템의 현재 상태와 원하는 상태를 일관성 있게 유지하고 추적하기 어려움
      - 각 컨테이너를 실행하기 위해 명령어를 사용하여 명령형 방식으로 직접 명령을 내려야 함.
      - 배포된 컨테이너의 상태를 변경하려면 다시 수동으로 명령을 실행하여 컨테이너를 중지하고 새로운 설정으로 다시 시작해야함.
        - ->> **컨테이너 오케스트레이션을 사용하면 무중단배포가 가능해짐.**
  - 쿠버네티스의 경우
    - 오토스케일링 : 컨테이너에 과부하가 발생할 경우 자동으로 컨테이너 인스턴스를 생성하여 확장하고 트래픽이 감소하면 불필요한 인스턴스를 제거하여 축소하여 효율적으로 리소스를 사용하는 오토스케일링 기능 제공.
    - 고가용성 : 애플리케이션을 여러 서버에 복제하여 하나의 서버에 장애가 발생해도 시스템이 계속 동작할 수 있게 함.
      - 이로 인해 클러스터는 항상 적절한 규모를 유지하며, 애플리케이션의 가용성과 신뢰성 향상.
    - 자동 복구 : 애플리케이션에 문제가 발생하면 자동으로 복구하거나 롤백 하는 기능을 갖추고 있음.
    - 선언적 구성 : 애플리케이션을 원하는 상태로 정의하고 그 상태를 지속적으로 유지.
      - 사용자가 YAML 파일을 사용하여 원하는 최종 상태를 정의하면 쿠버네티스 시스템은 원하는 상태와 현재 상태가 일치하는지 지속적으로 체크하고 해당 상태를 유지.
  - 참고 :
    - https://aws.amazon.com/ko/what-is/container-orchestration/
    - https://astronsec.com/gnuboard/blog/18
    - https://velog.io/@khn1346/컨테이너-도커-컨테이너-오케스트레이션

## 15주차
- AWS RDS를 활용하는 주요 이점과 EC2에 직접 데이터베이스를 설치하여 운영하는 것과 비교했을 때의 차별점에 대해 설명해주세요. 그리고 RDS를 사용하는 것이 적합하지 않을 수 있는 상황도 함께 언급해주세요.
  - 주요 이점
    - 자동 설치 제공
    - 보안패치 자동 업데이트
    - 자동 백업 지원
    - 모니터링 기본 제공
    - 클릭 몇 번으로 확장 가능
    - 운영 부담을 줄여줌
  - EC2 vs. RDS
    - EC2는 직접 관리 영역이 많음.
      - 개발자나 운영자가 데이터베이스 설치, 백업, 확장, 가용성 확보까지 모두 책임져야 함.
      - AWS가 대신 관리해주는 부분은 운영체제 설치/관리와 기반시설(하드웨어, 네트워크, 전력 등) 정도
      - -> 문제점
        - 백업 스크립트 직접 작성 필요
        - 장애 발생 시 수동 복구
        - 확장도 직접 서버를 증설해야 함
        - 운영 부담이 크고, 관리 포인트가 많음
    - RDS
      - 거의 모든 항목을 AWS가 관리
      - AWS가 데이터베이스 설치, 패치, 백업, 확장, 가용성 확보까지 책임져 줌
      - 사용자는 SQL 쿼리 작성, 데이터 모델링, 성능 최적화 같은 핵심 비즈니스 로직에만 집중하면 됨
      - 장점
        - 자동 백업 및 장애 복구 -> 운영자가 직접 관리하지 않아도 됨
        - 클릭 몇 번으로 확장 가능 -> 개발 생산성 향상
        - 고가용성 보장 -> 다중 AZ배포, 자동 페일오버 지원
        - 보안 및 패치 자동화 -> 최신 상태 유지
        - > (AZ = 가용 영역)/ 한 리전 안을 여러 개의 물리적으로 분리된 데이터 센터로 나눈 단위
        - > 장애가 발생했을 때, 시스템이 알아서 예비 서버로 전환되는 것 / 장애전환(failover)
  - RDS 한계
    - 운영 부담을 줄여주는 대신 비용이 발생함.
    - 단순한 테스트/학습 목적이라면 로컬 DB로 충분
    - DB에서 낮은 수준의 설정 (복잡한 커스터마이징)이 필요한 경우 RDS는 제약이 있을 수 있음
    - 운영 효율성과 안정성이 중요하다면 RDS
    - 커스터마이징, 실험 환경, 비용 절감이 필요하다면 EC2에 직접 설치하는 것이 더 적합할 수 있음

- GitHub Actions 워크플로우에서 사용할 수 있는 다양한 트리거(Trigger) 유형을 설명하고, 각 트리거 유형이 적합한 CI/CD 시나리오에 대해 설명하세요.
  - 트리거
    - push 이벤트 : 특정 브랜치에 코드가 push될 때 실행
      - CI: 새 커밋마다 자동으로 빌드 및 테스트 수행
      - CD: 배포를 위한 브랜치에 푸시되면 자동으로 배포 수행
    - pull_request 이벤트 : pr 생성/수정 시 실행
      - PR 검증 : 변경사항이 main에 병합되기 전에 해당 코드에 문제가 있는지 확인
    - schedule 이벤트 : 크론 표현식을 사용해 일정 주기로 실행
      - 야간 빌드/테스트 자동화
      - 백업/로그 정리를 위한 스크립트 실행
      - 정기적인 보안 스캔 또는 종속성 업데이트 확인
      - etc.. 일정 주기로 수행되어야 하는 작업
  - ```
    on:
      push:
        branches: [ "main", "develop" ]  # main, develop 에 푸시 시
      pull_request:
        branches: [ "main" ]  # main에 pr 생성 시
      schedule:
        - cron: '0 9 * * 1'   # 매주 월요일 오전 9시에 실행
    ```
  - 그 외 트리거 및 자세한 내용은 공식문서
    - [GitHub 액션 문서](https://docs.github.com/ko/actions/reference/workflows-and-actions/events-that-trigger-workflows)
  - 공식 문서 외 참고 :
    - https://velog.io/@khn1346/GitHub-Actions%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%EB%90%98%EB%8A%94-%ED%8A%B8%EB%A6%AC%EA%B1%B0Trigger-%EC%9C%A0%ED%98%95-mh1k5zir
   
---

## 19주차
#### 세션 기반 인증과 토큰 기반 인증의 차이점과 각각의 보안 고려사항에 대해 설명하세요.
  - 세션 기반
    - 세션 기반은 인증 정보를 서버쪽에 저장하는 방식으로 다시 말해 클라이언트(브라우저)가 가지고 있는 것이 아닌 호스팅하는 주체 서버쪽에서 인증 정보를 세션저장소에 저장하여 세션 ID를 통해 접근하는 방식이다.
    - 사용자 식별 정보나 인증 상태 정보,역할 등을 세션 저장소에 저장하고 세션ID를 통해 접근하게 된다.
    - 서버는 해당 세션ID를 set-cookie 헤더에 담아 클라이언트로 전달하게 되고 클라이언트는 해당 정보로 쿠키를 생성하고 요청 시 cookie헤더에 세션ID를 담아 서버에 인증하게 된다.
    - 보안 고려사항
      - 세션 ID는 충분히 긴 난수를 사용해야 하고 단순 증가하는 숫자나 시간 기반 ID는 공격자가 쉽게 추측할 수 있으므로 사용하면 안됩니다. 또한 동일한 세션 ID가 동시에 재발급되지 않도록 보장해야 합니다.
      - set-cookie헤더에 보안 속성을 추가하여 쿠키를 생성해야 합니다.
        - Secure : HTTPS 연결에서만 전송
        - HttpOnly : JavaScript로 쿠키 접근 차단 / 해당 옵션으로 XSS 공격을 완화합니다.
          - XSS : cross-site-Scripting / 사이트에 악의적인 스크립트를 넣는 기법
        - SameSite : 크로스 사이트 요청 제한 -> CSRF 완화
          - CSRF : cross-site request forgery / 사용자가 자신의 의지와 무관하게 공격자가 의도한 행동을 해서 공격하는 기법 -> 해당 계정의 비밀번호 등을 변경시켜서 해당 유저로 위장 같은 방법이 가능하다.
          - 옵션
            - Strict : 크로스 사이트 모든 내비게이션에서 쿠키 미전송 / 민감한 백오피스에서 사용한다 / 소셜 로그인, 외부 리디렉션 시 불편함
            - Lax : 대부분의 크로스 사이트 요청에서 미전송, 탑 레벨 GET 네비게이션은 예외 / 일반 웹앱 기본값이다.
            - None + Secure : 출처 상관없이 전송 / 제3자 쿠키 필요 시 / 반드시 Secure가 동반되어야 한다. 
      - 세션 만료 정책
        - 유휴 타임아웃 : 일정 시간 요청이 없으면 세션 자동 만료
        - 절대 타임아웃 : 세션이 생성된 지 일정 시간이 지나면 무조건 만료
        - 강제 로그아웃 : 관리자가 특정 사용자의 세션을 강제로 만료시킬 수 있어야 함
      - 공격자가 미리 발급받은 세션 ID를 피해자에게 강제로 사용하게 하면, 로그인 후에도 공격자가 동일 세션을 이용할 수 있음
        - 로그인 성공 시 새로운 세션 ID를 무조건 재발급하거나 권한 상승 시에도 세션 ID를 다시 생성하는 방식으로 해결 가능
       
      - 로그아웃 처리 기능을 만든다
        - 서버 측에서 세션 저장소에서 해당 세션 ID를 삭제하거나
        - 클라이언트 쿠키 만료 지시(Max-age=0 또는 Expires 속성) : 이를 통해 브라우저 쿠키를 제거한다.
        - 로그아웃 요청 시 즉시 세션이 무효화되어야 하며, 남아 있는 요청이 더 이상 인증되지 않아야 한다.

  - 토큰 기반
    - Authorization 헤더를 통해 토큰을 관리하고 서버로 전달한다.
    - 클라이언트가 토큰에 인증 정보를 포함하여 서버로 전달하기 때문에 서버는 별도로 상태를 저장하지 않는다.
    - 서버에서 토큰 자체를 관리하지 않기 때문에 보안성 측면에서 더 신중한 설계가 필요하지만 그렇기 때문에 확장성 면에서 매우 유리하다.
    - 토큰은 암호화되지 않은 사용자 정보를 포함할 수 있어 유출 시를 대비해 민감한 정보는 토큰에 넣지 않아야 한다.
      - 암호화하지 않기 때문에 HTTPS 필수
    - 토큰은 만료되기 전까지는 기본적으로 무효화가 불가능하다.
    - JWT(JSON Web Token)
      - 보통 2가지 종류의 토큰과 함께 사용 된다.
      - Access Token
        - API 요청 시 인증/인가 역할을 수행하며 유효 기간을 짧게 설정한다.
      - Refresh Toekn
        - Access Token 재발급을 목적으로 사용된다.
        - 유효 기간을 길게 설정하여 사용자 경험을 개선한다.
      - Access Token만 있으면 권한 요청이 가능하기 때문에 탈취될 경우를 대비해 짧은 만료 시간을 가지고 Refresh Token을 이용해 주기적으로 Access Token을 재발급한다. 이를 통해 보안을 강화하고 사용자 경험을 개선할 수 있다.
     
    - 보안 고려사항
      - JWT는 다음과 같은 점들을 고려해야 한다.
      - 민감한 정보 포함 금지
        - Base64URL로 인코딩되어 있을 뿐 암호화되어 있지 않기 때문에 민감 정보를 포함하지 않아야 함.
      - 토큰 만료 시간 설정
        - 만료 시간이 없는 토큰은 탈취될 경우 영구적으로 악용될 수 있음
      - HTTPS 사용
        - 네트워크를 통해 전달되므로 반드시 HTTPS를 통해 암호화된 채널에서만 전송해야 함
      - 저장 위치 주의
        - 로컬 스토리지는 XSS 공격에 취약하므로 대신 HttpOnly 쿠키 또는 보안이 강화된 세션 스토리지 사용을 권장한다.
      - 토큰 무효화 전략
        - 발급 후 만료 전까지는 무효화할 수 없어 이를 보완하기 위해 서버 측에서 블랙리스트를 관리하거나 Redis같은 인메모리 DB를 활용할 수 있음
        - Refresh 토큰의 경우 서버 DB에서 Refresh 토큰을 제거하거나 Refresh 토큰이 사용 될 때마다 새 Refresh 토큰을 발급하고 이전 것을 무효화하는 방식으로 탈취 여부를 확인할 수 있다.

#### OAuth 2.0의 주요 컴포넌트와 Authorization Code Grant 흐름을 설명하세요.
- **주요 컴포넌트**
  - Resource Owner
    - Resource의 실제 소유자
    - **서비스 사용자(User)를 의미한다**
    - 사용자는 자신의 계정 정보를 직접 관리하고 있으며, OAuth 2 프로세스에서 최종 결정권을 가진 주체
   
  - Client
    - Resource Owner를 대신하여 Resource에 접근하는 애플리케이션
    - 서버, 웹, 모바일 등 다양한 형태가 될 수 있음
    - 서비스를 이용하고자 하는 쪽이 Client
      - 즉, OAuth 서비스 제공자 (ex: google, naver, kakao)가 아닌 OAuth를 도입한 애플리케이션 (우리가 개발한 서버, 웹 등)이 Client
   
  - Resource Server
    - 보호된 자원을 실제로 저장하고 있는 서버
    - Client의 요청을 Access Token으로 검증하고, 올바르다면 자원을 반환
    - ex : Google Photo API 서버가 Resource Server
   
  - Authorization Server
    - Resource Owner의 인증을 처리하고, Client에게 Access Token을 발급하는 서버
    - OAuth 2에서 가장 중요한 역할을 맡고 있으며, 자격 증명의 신뢰성을 보장
    - Access Token 발급자
    - Access Token은 Resource Owner가 아닌 Client에 제공된다.
    - ex : Google의 로그인 서버
- **Authorization Code Grant**
  - 권한 부여 승인을 위해 자체 생성한 Authorization Code를 전달하는 방식
    - 가장 많이 쓰이고 기본이 되는 방식
  - Refresh Token을 사용할 수 있음
    - Implicit grant과 Client Credentials Grant는 불가능.
  - 권한 부여 승인 요청 시 응답 타입(response_type)을 `code`로 지정하여 요청
  - *인증 처리 흐름*
    - Resource Owner는 소셜 로그인 버튼을 누르는 등의 서비스 요청을 Client(애플리케이션)에게 전송
    - Client는 Authorization Server에 Authroization Code를 요청
      - 이때 미리 생성한 Client ID, Redirect URI, 응답 타입을 함께 전송
    - Resource Owner는 로그인 페이지를 통해 로그인 진행
    - 로그인이 확인되면 Authorization Server는 Authroization Code를 Client에게 전달
      - 이 전에 요청과 함께 전달한 Redirect URI로 Code 전달
    - Client는 전달 받은 Authorization Code를 이용해 Access Token 발급을 요청. Access Token을 요청할 때 미리 생성한 Client Secret, Redirect URI, 권한 부여 방식, Authorization Code를 함께 전송
    - 요청 정보를 확인한 후 Redirect URI로 Access Token 발급
    - Client는 발급받은 Access Token을 이용해 Resource Server에 Resource를 요청
    - Access Token을 확인한 후 요청 받은 Resource를 Client에게 전달

## 20주차
#### Spring 기반 웹 애플리케이션에서 발생할 수 있는 4가지 주요 보안 공격 (CSRF, XSS, 세션 고정, JWT 탈취)에 대해 설명하고, 각각에 대한 Spring Security 또는 일반적인 대응 전략을 설명하세요.
+ CSRF
  + 사용자가 의도하지 않은 요청을 공격자가 위조하여 서버에 보내도록 하는 공격 기법
  + 피해자는 이미 로그인된 세션을 가진 상태이므로, 서버는 공격자가 보낸 요청을 정상 요청으로 오인하게 된다.
  + Spring Security CSRF 보호 기본 메커니즘
    + 기본적으로 상태 유지 세션 기반 애플리케이션에서 CSRF 보호를 활성화
    + 서버는 요청 시 CSRF 토큰을 검증하여, 클라이언트가 의도적으로 요청을 보냈는지 확인
    + CsrfFilter
      + 모든 요청에 대해 CSRF 토큰을 검증하는 필터. 사용자가 보낸 요청에 _csrf 파라미터나 헤더 값이 없는 경우 AccessDeniedException 발생
    + CsrfToken
      + 서버가 세션별로 생성하여 클라이언트에 전달하는 토큰
      + 이 토큰은 매 요청마다 클라이언트가 전달해야 하며, 서버는 저장된 토큰과 비교하여 일치 여부를 검사합니다.
      + 토큰 생성 과정
        + 사용자가 최초로 세션을 생성하거나 페이지를 요청할 때 고유한 난수 기반 토큰 생성
        + `HttpSessionCsrfTokenRepository`(기본 구현체) 또는 `CookieCsrfTokenRepository`에 저장
      + 토큰 전달 방식
        + 폼 hidden 필드 : 서버에서 렌더링 시 _csrf 값을 자동 삽입
        + AJAX/SPA : 응답 헤더 또는 meta 태그로 전달 후, 요청 시 X-CSRF-TOKEN 헤더에 포함
        + Double-submit 쿠키 : 토큰을 쿠키와 헤더 모두에 담아 서버에서 일치 여부 검증
      + 검증 방식
        + 요청이 들어오면 CsrfFilter가 동작
        + 요청에 포함된 토큰(_csrf 필드 또는 헤더 값)을 추출
        + 저장소(HttpSession 또는 Cookie)에 보관된 토큰과 비교
        + 일치하면 정상 요청, 불일치 또는 누락 시 AccessDeniedException 발생
    + 보강 전략
      + Origin/Referer 검증 : 요청 헤더로 동일 출처인지 확인
      + SameSite 쿠키 : Lax 또는 Strict 설정으로 크로스 도메인 요청 차단
      + XSS 방어 : 토큰 탈취 방지를 위해 필수
+ XSS (Cross-Site Scripting)
  + 공격자가 악의적인 스크립트를 웹 페이지에 삽입하여 사용자의 브라우저에서 실행되도록 하는 공격 기법
  + 주로 쿠키 탈취, 세션 하이재킹, 악성 사이트 리다이렉션, 피싱 등에 활용
  + 주요 유형
    + Stored XSS : 서버 DB에 저장된 악성 스크립트가 여러 사용자에게 전달 됨
    + Reflected XSS : URL 파라미터에 삽입된 스크립트가 즉시 응답에 반영
    + DOM 기반 XSS : 서버 응답과 무관하게 클라이언트 JS 코드가 DOM 조작으로 공격 실행
  + Spring XSS 방어
    + 브라우저가 응답 헤더를 근거로 위험한 리소스/스크립트를 차단하도록 하는 접근
    + CSP 와 X-Content-Type-Options: nosniff
    + CSP (Content-Security-Policy) :
      + 스크립트/리소스 로딩,실행 원천 제한
      + 브라우저가 강제하는 보안 정책
        + 어떤 출처의 리소스만 로드/실행 가능한지 선언적으로 헤더에 규정
        + 스크립트 실행 원천을 제한
        + 인라인 스크립트 금지 또는 허용된 nonce/hash만 실행
      + nonce : 요청마다 서버가 난수 생성 -> `<script nonce="<값>">`에만 실행 허용
        + 동적 스크립트에 유연
        + 응답마다 생성/주입 필요
        + 서버사이드 렌더링(Thymeleaf 등)에서 사용 권장
      + hash : 스크립트 내용의 SHA-256 등 고정 해시가 정책에 등록된 것만 실행 허용
        + CDN/정적 스크립트에 적합
        + 내용 변경 시 해시 갱신 필요
        + 빌드 파이프라인 연계에서 사용 권장
    + X-Content-Type-Options :
      + MIME 스니핑 방지
        + 잘못된 타입의 리소스 실행/렌더링 금지
        + JSON,스크립트 오인 실행 방지
      + 컨트롤러에서 정확한 Content-Type 지정
      + 전역으로 X-Content-Type-Options: nosniff 적용 → 브라우저 임의 추측 금지
    + CSP 위반 : 스크립트 / 리소스 차단
    + 타입 불일치 & nosniff : 렌더링 / 실행 거부
+ 세션 고정
  + 공격자가 미리 발급받은 세션 ID를 피해자에게 강제로 사용하게 한 뒤, 해당 세션을 탈취하는 공격 기법
  + 피해자가 정상적으로 로그인하더라도 이미 공격자가 알고 있는 세션 ID가 그대로 유지된다면 공격자는 세션을 가로챌 수 있음
  + Spring 세션 고정 보호 전략
    + 로그인 시점에 세션 ID를 어떻게 다룰지 결정하는 session fixation 보호 전략 제공
      + none
        + 기존 세션 그대로 유지
      + newSession
        + 새로운 세션을 생성하고 기존 속성은 복사하지 않음
        + 세션 완전 초기화
      + migrateSession
        + 새로운 세션을 생성하고 속성을 복사
        + 보안 + 사용자 편의 균형
      + changeSessionId
        + 세션 ID만 새로 발급, 속성 그대로 유지
        + Servlet 3.1+ 환경 권장
+ JWT 탈취
  + XSS공격과 CSRF 공격 등을 통해 공격자는 사용자의 쿠키나 로컬 스토리지에 접근하여 토큰을 탈취하는 방식
  + 액세스 토큰은 유효기간이 짧아 탈취당해도 상대적으로 안심할 수 있다. 그러나 리프레쉬 토큰의 경우 유효기간이 길기 때문에 안심할 수 없다.
    + 리프레쉬 토큰으로 얼마든 액세스 토큰을 발급 받아서 악의적으로 이용할 수 있기 때문
  + 리프레쉬 토큰 탈취 방식
    + 중간자 공격
      + 공격자가 사용자의 인터넷 서버와 해당 인터넷 트래픽의 목적지 사이에 끼어들어 데이터 전송을 가로채는 기법
      + 리프레쉬 토큰도 중간게 가로챌 수 있다. 중간자 공격자가 HTTPS 연결을 도청하여 헤더에 담긴 리프레쉬 토큰을 탈취
      + 대표적 중간자 공격 : 패킷 스니핑, 패킷 인젝션, 세션 하이재킹, SSL 스트리핑이 있음.
  + 중간자 공격 방지
    + VPN
    + HTTPS만 사용
    + 모든 통신에 종단간 암호화 실시
    + 출처가 불분명한 링크 클릭 X
    + 집/회사의 네트워크 보안 확인
  + +내용
    + 액세스 토큰과 리프레쉬 토큰을 사용
    + 액세스 토큰
      + API 요청 시 인증/인가 용 토큰
      + 유효 기간이 짧음
      + 메모리/스토리지에 보관
    + 리프레쉬 토큰
      + 액세스 토큰 재발급 목적
      + 유효 기간이 김
      + 안전하게 보관해야 한다.
        + 쿠키(HttpOnly + Secure + SameSite)
        + 장기간 보관되므로 DB 저장 후 화이트리스트/블랙리스트 방식으로 관리하는 경우도 많음
      + 무효화 전략
        + 서버 DB에서 Refresh 토큰을 제거하는 강제 로그아웃
        + Refresh 토큰 사용 시마다 새 Refresh 토큰을 발급하고 이전 것은 무효화하는 토큰 로테이션
+ 참고 :
  + https://velog.io/@ckdwns9121/토큰token은-어떻게-탈취-당하는가
#### JWT(JSON Web Token)의 구조와 각 구성 요소가 어떤 역할을 하는지 구체적으로 설명하세요.
+ **JWT 구조**
  + Header, Payload, Signature로 구성되어 있으며 각 부분은 `.`으로 구분됨
  + Header : 어떤 알고리즘으로 서명했는지, 어떤 타입의 토큰인지 정의
    + Base64URL로 인코딩
      ```json
      {
        "typ": "JWT", // 토큰의 타입 JWT
        "alg": "HS256"  // 해싱 알고리즘 지정
      }
      ```
  + Payload : 사용자 정보와 클레임 데이터가 담김
    + Base64URL로 인코딩
    + 사용자의 고유 정보와 권한, 토큰 발급 시간 등이 포함
    + Base64URL 인코딩이라 민감한 정보를 직접 담아서는 안됨 (암호화가 아님)
    + 클레임의 종류
      + 등록된(rgistered) 클레임
      + 공개(public) 클레임
      + 비공개(private) 클레임
    + 등록된 클레임
      + 서비스에서 필요한 정보들이 아닌, 토큰에 대한 정보를 담기 위하여 이름이 이미 정해진 클레임들
      + 모든 클레임의 사용은 선택적임.
    + 공개 클레임
      + 충돌이 방지된 이름을 가지고 있어야 함
      + 충돌을 방지하기 위해서 이름을 URI 형식으로 지음.
    + 비공개 클레임
      + 등록된 클레임도 아니고, 공개된 크레임도 아님
      + 클라이언트와 서버 간 협의 하에 사용되는 클레임 이름들
      + 이름이 중복되어 충돌이 될 수 있음.
    ```json
    {
        "iss": "velopert.com",
        "exp": "1485270000000",
        "https://velopert.com/jwt_claims/is_admin": true,
        "userId": "11028373727102",
        "username": "velopert"
    }
    ```
  + Signature : 헤더와 페이로드가 변조되지 않았음을 검증
    + 서버의 비밀 키로 생성됨
    + 클라이언트가 보낸 토큰이 변조되지 않았음을 검증
    + 헤더의 인코딩 값, 정보의 인코딩 값을 합친 후 주어진 비밀키로 해쉬하여 생성
    ```
    HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      secret)
    ```
+ 참고 :
  + https://velopert.com/2389

---

## 21주차
#### 멀티스레드 환경에서 발생하는 대표적인 문제 중 하나인 경쟁 상태(Race Condition)에 대해 설명하고, 이를 해결하기 위한 다양한 전략을 설명해보세요.
+ 경쟁 상태
  + 둘 이상의 스레드나 프로세스가 공유된 자원(ex: 변수, 메모리 등)에 대해 동시에 접근하는 변경하는 경우 발생할 수 있는 문제
  + 어떤 스레드나 프로세스가 먼저 접근하여 변경한 값을 다른 스레드나 프로세스가 무시하거나 올바르지 않은 값을 사용하는 등의 문제가 발생할 수 있음
 
+ 해결 전략
  + Synchronized
    + 여러 스레드가 공유 자원에 접근할 때, 한 번에 하나의 스레드만 접근하도록 제한하는 방식
  + 불변 객체 활용
    + 객체의 상태를 한 번 정의한 후 변경할 수 없도록 설계
    + 상태 변경이 필요한 경우, 기존 객체를 수정하지 않고 새 객체를 생성
  + 스레드 로컬 사용
    + 각 스레드에 독립된 변수를 제공하여 공유 자체를 하지 않도록 설계
    + 공유 자원을 없애 경쟁 상태를 원천적으로 방지
  + 원자적 연산 사용
    + 하나의 연산이 중단 없이 실행되도록 보장하는 방식
    + 내부적으로 CAS(Compare-And-Swap)를 활용해 락 없이 동시성 처리
  + Concurrent 자료구조 활용
    + 자바에서 제공하는 동시성 컬렉션을 사용해 직접 동기화하지 않고도 안전한 접근을 가능하게 함
    + 내부적으로 세분화된 락 또는 비동기 처리 구조로 구성됨
+ 참고 :
  + https://velog.io/@uurr/%EB%A9%80%ED%8B%B0%EC%8A%A4%EB%A0%88%EB%93%9C-%ED%99%98%EA%B2%BD%EA%B3%BC-%EA%B2%BD%EC%9F%81-%EC%83%81%ED%83%9CRace-Condition-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%ED%95%B4%EA%B2%B0-%EC%A0%84%EB%9E%B5#%EA%B2%BD%EC%9F%81-%EC%83%81%ED%83%9C-%ED%95%B4%EA%B2%B0-%EC%A0%84%EB%9E%B5
 
> 경쟁 상태와 교착상태의 차이
>> 경쟁 상태 : 두 개 이상의 스레드나 프로세스가 같은 자원에 동시에 접근하여 실행 순서에 따라 결과가 달라지는 현상\
>> 교착 상태 : 두 개 이상의 스레드가 서로가 가진 자원을 기다리며 무한히 대기하는 상태

#### 비동기 환경에서 MDC(Logback Mapped Diagnostic Context)나 SecurityContext 같은 컨텍스트 정보를 스레드 간에 전달해야 할 경우, 처리하는 방법에 대해 설명하세요.
+ > Spring 애플리케이션에서는 로그 추적을 위해 MDC를 사용하거나, 인증 정보를 담는 SecurityContext를 다루는 경우가 많음\
  > 이 두 가지는 기본적으로 ThreadLocal 기반으로 동작하기 때문에, @Async, 스레드 풀, Reactor 같은 비동기 실행 환경에서는 새로운 스레드로 전환될 때 값이 사라져 버림.\
  > 따라서 추가적인 전파 작업을 해주지 않으면 로그에 요청 ID가 누락되거나, 인증 정보가 비동기 실행 코드에서 보이지 않는 문제가 발생할 수 있음 
+ Spring Security에서는 DelegatingSecurityContext 계열 유틸리티를 제공
  + DelegatingSecurityContextRunnable
  + DelegatingSecurityContextCallable
  + DelegatingSecurityContextExecutor
  + DelegatingSecurityContextAsyncTaskExecutor
  + -> 비동기 실행 중에도 SecurityContext가 자동으로 복사되어 인증 정보가 유지 됨

+ MDC는 스레드별 저장소
+ MDC를 비동기 환경에서 사용하려면 TaskDecorator를 통해 부모 스레드의 컨텍스트를 복사해 주어야 함
  + ```java
    @Bean
    public TaskDecorator mdcTaskDecorator() {
        return runnable -> {
            Map<String, String> contextMap = MDC.getCopyOfContextMap();
            return () -> {
                if (contextMap != null) MDC.setContextMap(contextMap);
                try {
                    runnable.run();
                } finally {
                    MDC.clear();
                }
            };
        };
    }
    ```
  + -> 스레드 풀 내부에서 실행되는 작업도 부모 스레드의 MDC 값을 그대로 이어 받을 수 있음.

+ 참고 :
  + https://velog.io/@jinsol8k/%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%8A%A4%EB%A0%88%EB%93%9C-%EA%B0%84-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8-%EC%A0%95%EB%B3%B4-%EC%A0%84%EB%8B%AC

---
## 22주차
#### Spring Cache에서 @Cacheable, @CachePut, @CacheEvict의 차이점과 각각을 어떤 상황에서 사용하는 것이 적절한지 설명해주세요.
+ **@Cacheable**
  + 메서드 실행 결과를 캐시에 저장하고
  + 동일한 인자로 호출 시 저장된 값을 반환
  + ```@Cacheable("products")` -> 캐시 이름이 `products`인 저장소에 결과를 저장
  + 동일한 ID로 다시 호출되면 DB 접근 없이 캐시된 결과를 반환
  + `condition`과 `unless` 속성을 이용하여 캐시 조건을 제어 할 수 있음
    + `condition` : 캐싱을 수행할 조건 지정 (true일 때만 저장)
    + `unless` : 메서드 실행 후 결과에 따라 저장 여부 결정
    + `@Cacheable(value = "products", condition = "#id > 0", unless = "#result == null")`
  + 속성
    + value : cacheName의 Alias
    + key : 동일한 cache name을 사용하지만 구분될 필요가 있을 때 사용
    + condition : 캐싱을 수행할 조건 지정. true일 때 캐싱 적용
    + unless : 캐싱을 막기위해 사용. 메서드 실행 후 결과에 따라 저장 여부 결정. true일때 캐싱하지 않음
+ **@CachePut**
  + 항상 메서드를 실행하고, 실행 결과를 캐시에 강제로 갱신
    + @Cacheable의 경우 캐시에 없을 때만 실행 / @CachePut은 한상 실행
    + @Cacheable은 조회 성능 향상의 목적 / @CachePut은 데이터 갱신 후 캐시 자동화의 목적
  + 속성
    + value
    + key
    + condition
+ **@CacheEvict**
  + 캐시된 데이터를 제거할 때 사용
  + key : 특정 데이터 / allEntries : 전체 삭
  + `beforeInvocation = true` : 메서드 실행 전에 캐시 삭제 / Default : false
  + 속성
    + value
    + key
    + condition
    + allEntries : 캐시 내의 모든 리소스를 삭제할 지 여부
 + **적절한 상황**

| 어노테이션     | 설명                                    | 사용 상황         |
|-------------|----------------------------------------|----------------|
| @Cacheable  | 결과를 캐시에 저장하도고 동일 인자 시 캐시에서 반환 | 조회 시 캐시에 저장 |
| @CachePut   | 메서드 실행 후 캐시를 강제 갱신                | 수정 시 캐시 갱신  |
| @CacheEvict | 캐시 제거                                 | 삭제 시 캐시 제거  |

##### 추가 내용
+ 여러 캐시 동작을 묶어서 수행 가능
  + **@Caching**
  + ```java
    @Caching(
        put = {@CachePut(value = "products", key = "#product.id")},
        evict = {@CacheEvict(value = "products", key = "'list'")}
    )
    ```
+ 클래스 단위에서 공통 캐시 이름, 키 생성 정책 지정
  + **@CacheConfig**
  + ```java
    @Service
    @CacheConfig(cacheNames = "products")
    public class ProductService {...}
    ```
+ SpEL 사용 가능.
  + SpEL 활용하여 캐시 키를 동적으로 지정 가능
 
> 캐시 적용 고려 사항
>> 1. 자주 조회되지만 자주 변경되지 않는 데이터에 효과적 \
>> 2. 캐시의 용량 관리 및 TTL 설정 \
>>> 일정 수준 이후에는 캐시 크기를 키워도 효과가 미미


#### 로컬 캐시와 분산 캐시의 개념 차이와 각각의 장단점, 그리고 실무에서 어떤 기준으로 선택해야 하는지 설명해주세요.
+ 로컬 캐시
  + 애플리케이션 내부 메모리 (RAM) 에 데이터를 저장하는 캐시 구조
    + 서버 자체에 존재하므로 외부 네트워크 요청 없이 가장 빠르게 데이터에 접근할 수 있음
    + 장점 :
      + 네트워크 호출이 없어 응답 속도가 가장 빠름
      + 단순 구현 가능
      + 외부 장애에 영향을 받지 않음
    + 단점 :
      + 서버마다 캐시가 달라 데이터 불일치 가능성 있음
      + 서버 재시작 시 캐시가 모두 초기화 됨
      + 확장성이 낮음
    + *소규모 서비스, 설정값 캐시에 주로 사용*
+ 분산 캐시
  + 여러 서버 인스턴스가 동일한 캐시 서버를 공유하는 구조
  + 일반적으로 Redis, Memcached 같은 인메모리 서버를 사용
  + 여러 서버가 하나의 중앙 캐시 저장소를 공유
  + 데이터 일관성 유지가 용이하며, 확장성에 유리
  + 장점 :
    + 여러 서버 간 데이터 공유 가능
    + 캐시 서버 장애 시에도 Persistence(데이터 영속화) 설정 가능
    + 수평 확장(Scale-Out)에 유리
  + 단점 :
    + 네트워크 지연이 존재 함
    + Redis 장애 시 모든 요청이 DB로 몰릴 수 있음
    + 운영/모니터링 복잡도가 증가 함
  + 적합한 사용 사례
    + 여러 서버에서 동일 사용자 세션 공유 시
    + 트래픽이 많은 상품 목록, 상세 조회 캐싱 시
    + 갱신 주기가 짧은 데이터 관리 시
    + *암호화 되지 않은 민감 정보 저장은 비추천*

> 로컬 캐시와 분산캐시를 동시에 사용할 수는 없는가?
>> 다중 레벨 캐시\
>> 로컬 캐시와 분산 캐시 구조를 결합한 형태\
>> 로컬 캐시에서 데이터 검색 -> 없으면 분산 캐시로 이동\
>> 분산 캐시에서도 없으면 DB에서 조회 후 로컬,분산 모두에 저장

> 분산 환경에서 로컬 캐시의 정합성 보장
>> 로컬 캐시는 각 WAS에서 데이터를 캐싱하고 있는 것이기 때문에 1번 WAS를 통해 원본 데이터에 수정이 발생할 경우 다른 WAS 들의 캐시 데이터에도 이를 적용 해줘야 함.\
>> TTL 설정.
>>> 캐싱되어 있는 데이터가 유효한 시간을 정해주는 방법. TTL이 지난 데이터는 캐시에서 사라지고 추후 조회 요청이 발생하면 최신 데이터를 조회하여 캐시를 채우는 방식\
>>Invalidation Message Propagation
>>> 캐시 데이터에 수정이 발생할 경우 기존 캐시 데이터는 유효하지 않다는 것을 다른 WAS에 전파시켜 기존 캐시 데이터를 무효화시키는 방법\
>>> 데이터의 변경 주기가 일정하지 않고 유저 액션에 의해 일어나는 경우 적절

> 분산 시스템에서 로컬 캐시활용
>> 데이터베이스 데이터 갱신 시 : 데이터베이스에 변경이 발생하면 Redis에 해당 데이터를 업데이트가 있음을 발행\
>> Redis Pub/Sub 알림 : Redis는 데이터 변경 이벤트를 모든 구독자(애플리케이션 서버)에 전파\
>> 로컬 캐시 동기화 : 애플리케이션 서버는 Redis의 이벤트를 받아 로컬 캐시를 무효화\
>> **자주 변화하지 않는 데이터에 대한 잦은 조회 발생 시 효과적**\
>> **최종적 일관성(Eventual Consistency)**

- 참고 : 
  + [분산 환경에서 로컬 캐시의 정합성 보장](https://00h0.tistory.com/112)
  + [분산 시스템에서 로컬 캐시 활용 / 카카오페이 기술블로그](https://tech.kakaopay.com/post/local-caching-in-distributed-systems/)

## 23주차
#### TCP/IP 4계층 모델과 OSI 7계층 모델에 대해 각각 설명하고, 두 모델을 비교해보세요.
<img width="687" height="565" alt="image" src="https://github.com/user-attachments/assets/b8408b19-4880-48db-b95f-8d5bf3cc1e67" />

- **OSI 7계층**
  - **1계층(물리 계층)**
    - 전송 단위 : Bit
    - 장비 : 케이블, 허브
    - 전기적 신호가 나가는 물리적인 장비
      - 이 계층에서는 단지 데이터를 전달할 뿐, 전송하려는(받으려는) 데이터가 무엇인지, 어떤 에러가 있는지 등에 대해서는 신경쓰지 않는다.
      - 단지 데이터를 전기적인 신호로 변환해서 주고받는 기능만 있을 뿐
     
  - **2계층(데이터 링크 계층)**
    - 전송 단위 : 프레임(Frame)
    - 장비 : 브릿지, 스위치, 이더넷
    - 물리계층을 통해 송수신되는 정보의 오류와 흐름을 관리하여 안전한 정보의 전달을 수행할 수 있도록 도와주는 역할
      - 따라서 통신에서의 오류도 찾아주고 재전송도 하는 기능을 가지고 있음.
      - 이 계층에서는 맥 주소를 가지고 통신하게 됨
      - 데이터 링크 계층은 포인트 투 포인트 간 신뢰성 있는 전송을 보장하기 위한 계층으로 CRC기반의 오류 제어와 흐름 제어가 필요함
     
  - **3계층(네트워크 계층)**
    - 전송 단위 : 패킷
    - 경로(Route)와 주소(IP)를 정하고 패킷을 전달해주는 것이 이 계층의 역할
      - 목적지까지 가장 안전하고 빠르게 데이터를 보내는 기능을 말함. 따라서 최적의 경로를 설정해야 함
      - 이런 라우팅 기능을 맡고 있는 계층이 네트워크 계층
     
  - **4계층(전송 계층)**
    - 전송 단위 : 세크먼트
    - 양 끝단의 사용자들 간의 신뢰성 있는 데이터를 주고 받게 해주는 역할
      - 송신자와 수신자 간의 신뢰성있고 효율적인 데이터를 전송하기 위하여 오류검출 및 복구, 흐름제어와 중복검사 등을 수행.
      - 데이터 전송을 위해서 Port 번호가 사용됨. 대표적인 프로토콜로는 TCP와 UDP
     
  - **5계층(세션 계층)**
    - 응용 프로세스가 통신을 관리하기 위한 방법을 정의
    - 이 계층은 TCP/IP 세션을 만들고 없애는 역할을 하고 있음
   
  - **6계층(표현 계층)**
    - 전송하는 데이터의 표현방식을 결정 (ex: 데이터변환, 압축, 암호화 등)
    - GIF,JPEG,ASCII 등
    - 표현 계층 3가지 기능
      - 송신자에서 온 데이터를 해석하기 위한 응용계층 데이터 부호화, 변화
      - 수신자에서 데이터의 압축을 풀 수 잇는 방식으로 된 데이터 압축
      - 데이터의 암호화와 복호화
    - 인코딩이나 암호화 등의 동작이 표현계층에서 이루어짐
   
  - **7계층(응용 계층)**
    - 사용자와 가장 가까운 계층이 바로 응용 계층
    - 우리가 사용하는 응용 서비스나 프로세스가 바로 응용계층에서 동작
    - 대표적으로 우리가 잘 알고있는 HTTP, FTP 등의 프로토콜이 응용 계층이 속함

- 참조 :
  - https://inpa.tistory.com/entry/WEB-🌐-OSI-7계층-정리

- 네트워크 전송 시 데이터 표준을 정리한 것이 OSI 7계층 이라면, 이 이론을 실제 사용하는 인터넷 표준이 TCP/IP 4계층
  - 쓸데없이 복잡한 OSI 7계층을 4-5계층으로 분류하여 적용한 것
 
- **TCP/IP 4계층**
  - **4계층(응용 계층)**
    - **OSI 7계층에서 5,6,7 계층** / **HTTP / FTP 담당**
    - 데이터 단위 : 데이터 / 메시지
    - 사용자와 가장 가까운 계층
      - 사용자-소프트웨어 간 소통을 담당하는 계층. 웹 프로그래밍을 하면서 흔히 접하는 여러 서버나 클라이언트 관련 응용 프로그램들이 동작하는 계층.
      - 주로 응용 프로그램들끼리 데이터를 교환하기 위한 계층
    - ex: 파일 전송, 이메일, FTP, HTTP, DNS, SMTP 등

  - **3계층(전송 계층)**
    - **OSI 7계층에서 전송 계층** / **TCP / UDP 담당**
    - 데이터 단위 : 세그먼트
    - 전송 주소 : Port
    - 통신 노드 간의 데이터 전송 및 흐름에 있어 신뢰성을 보장
      - 다시 말해 데이터를 적절한 어플리케이션에 제대로 전달되도록 배분함을 의미. 다른 말로 End-to-End의 신뢰성을 확보한다고 표현.
      - 대표적인 프로토콜로 TCP와 UDP가 있음. TCP는 연결 지향형 프로토콜로 패킷에 하나의 오류라도 있으면 재전송을 위해 에러를 복구. UDP는 패킷을 중간에 잃거나 오류가 발생하도 이에 대처하지 않고 계속해서 데이터를 전송하는 TCP에 비해 간단한 구조를 가지는 프로토콜
    - ex: TCP, UDP 등

  - **2계층(인터넷 계층)**
    - **OSI 7계층에서 네트워크 계층** / **IP 담당**
    - 데이터 단위 : 패킷
    - 전송 주소 : IP
    - 네트워크 상에서 데이터의 전송을 담당하는 계층.
      - 서로 다른 네트워크 간의 통신을 가능하게 하는 역할을 수행 (연결성 제공)
      - 단말을 구분하기 위해 논리적인 주소로 IP 주소를 할당하게 되고 이 IP 주소로 네트워크 상의 컴퓨터를 식별하여 주소를 지정할 수 있도록 해준다.
      - 네트워크끼리 연결하고 데이터를 전송하는 기기인 '라우터'라고 하며, 라우터에 의한 네트워크 간의 전송을 '라우팅'이라고 한다. 이 라우터가 내부의 라우팅 테이블을 통해 경로 정보를 등록하여 데이터 전송을 위한 최적의 경로를 찾는데, 이렇게 출발지와 목적지 간의 데이터 전송 과정을 가리켜 End-to-End 통신이라고 부른다.
    - ex: IP, ARP, ICMP, RARP

  - **1계층(네트워크 연결 계층)**
    - **OSI 7계층에서 물리+데이터링크 계층**
    - 데이터 단위 : 프레임
    - 전송 주소 : MAC
    - 물리적인 데이터의 전송을 담당하는 계층.
      - 여기서는 인터넷 계층과 달리 같은 네트워크 안에서 데이터가 전송됨. 노드 간의 신뢰성 있는 데이터 전송을 담당하며, 논리적인 주소가 아닌 물리적인 주소인 MAC을 참조해 장비간 전송을 하고, 기본적인 에러 검출과 패킷의 Frame화를 담당
    - ex: MAC, LAN, 패킷망 등에 사용되는 것(대포적으로 Ethernet)

- 추가내용
  - **OSI 7계층의 장점**
    - 모듈화
      - 각 계층은 독립적으로 설계되어 있으므로 하나의 계층을 수정하더라도 다른 계층에는 영향을 주지 않음
    - 표준화
      - OSI 모델은 국제 표준화 기구에서 정의하고 있으므로 다양한 시스템과 기기 간의 호환성을 보장
    - 간소화
      - 각 계층은 자신의 역할에만 집중하므로 복잡한 네트워크 통신을 단순화할 수 있음
  - **네트워크 통신 과정을 분리하는 이유**
    - *네트워크의 안정성, 신뢰성, 유지보수성을 위해서*
    > 각 계층을 분리함으로써 개발 및 유지보수가 용이해짐. 문제 발생 시 문제가 발생한 계층에서 해결이 가능해지기 때문에 \
    > 각 계층은 자신의 책임 범위 내에서 독립적으로 동작하기 때문에, 문제가 발생했을 때 문제가 발생한 계층에서 해결이 가능 \
    > 이는 문제를 진단하고 해결하는 데 시간을 단축시키고,전체 시스템의 안정성과 신뢰성을 높일 수 있음 \
    > 또한, 계층적인 구조를 가짐으로써 각 계층이 추상화되고 인터페이스를 정의할 수 있음. 계층 간 인터페이스를 통해 각 계층은 자신의  책임 범위 내에서만 통신하게 되어, 다른 계층에 영향을 미치지 않으면서 새로운 기술이나 장비를 도입하기 쉬워짐
    
- 참조 :
  - https://velog.io/@dyunge_100/Network-TCPIP-4계층에-대하여
  - https://inpa.tistory.com/329
  - https://mundol-colynn.tistory.com/167

#### 전송 계층에서 TCP와 UDP의 차이점은 무엇이며, 각각 어떤 상황에서 사용하는 것이 적절한가요?
- **TCP - 전송 제어 프로토콜**
  - IP규칙으로만 통신하기에 부족하거나 불안정하던 여러 단점들(패킷 순서가 이상하거나 패킷이 유실)을 커버해, 패킷 전송을 제어하려 신뢰성을 보증하는 프로토콜
  - IP규칙에 써있는대로 목적지까지 다다랐으면, TCP 규칙에 써있는대로 올바르게 도착했는지 정확히 누구에게 전달되야하는지 하나하나 따진다고 생각하면 된다.
  - 그래서 은행 업무나 메일과 같은 반드시 수신자가 정보를 받아야하는 **신뢰성 있는 통신**이 필요할 때 사용된다.
  - 특징
    - 연결 지향 방식으로 패킷 교환 방식을 사용(가상 회선 방식이 아님)
    - 3-way handshaking 과정을 통해 연결을 설정하고 4-way handshaking을 통해 해제함.
    - 흐름 제어 및 혼잡 제어
    - 높은 신뢰성을 보장
    - UDP보다 속도가 느림
    - 전이중(full-Duplex), 점대점(point to point) 방식
   
  - TCP 서버 특징
    - 서버소켓은 연결만을 담당
    - 연결과정에서 반환된 클라이언트 소켓은 데이터의 송수신에 사용된다
    - 서버와 클라이언트는 1대1로 연결된다.
    - 스트림 전송으로 전송 전송 데이터의 크기가 무제한
    - 패킷에 대한 응답을 해야하기 때문에(시간 지연, CPU 소모) 성능이 낮다
    - streaming 서비스에 불리하다(손실된 경우 재전송 요청을 하므로)
   
  - 3-way handshaking / 4-way handshaking
  > 3-way Handshaking
  >> 연결을 시작할 때 사용되며, 클라이언트와 서버 간에 SYN, SYN-ACK, ACK 패킷을 교환하여 연결을 확립
  
  > 4-way Handshaking
  >> 연결을 종료할 때 사용되며, FIN, ACK, FIN, ACK 패킷을 교환하여 연결을 종료 \
  >> 이 과정을 통해 양쪽 모두 데이터 전송이 완료되었음을 확인하고, 연결을 안전하게 종료할 수 있기 때문
  
- **UDP - 사용자 데이터그램 프로토콜**
  - 데이터그램 방식을 사용하는 프로토콜. 각각의 패킷 간의 순서가 존재하지 않는 독립적인 패킷을 사용
  - 데이터그램 방식은 목적지만 정해져있다면 중간 경로는 어딜 타든 신경쓰지 않기 때문에 핸드쉐이크 과정 같은 연결 설정이 필요 없게 된다.
    - UDP는 신뢰성을 확보하기 위해 거치던 TCP의 과정을 거치지 않기 때문에 속도가 더 빠를 수 밖에 없다.
    - 그래서 UDP는 **실시간 영상 스트리밍과 같은 고용량 데이터를 다루는 곳**에 이용된다.
   
  - UDP 특징 :
    - 비연결형 서비스로 데이터그램 방식을 제공한다.
    - 정보를 주고 받을 때 정보를 보내거나 받는다는 신호절차를 거치지 않는다.
    - UDP해더의 CheckSum 필드를 통해 최소한의 오류만 검출한다.
    - 신뢰성이 낮다.
    - TCP 보다 속도가 빠르다.
   
  - UDP 서버의 특징
    - 연결 자체가 없어서(connect 함수 불필요) 서버 소켓과 클라이언트 소켓의 구분이 없다.
    - 소켓을 활용해 IP와 Port를 기반으로 데이터를 전송
    - 서버와 클라이언트는 1대1, 1대n, n대m 등으로 연결될 수 있다.
    - 데이터그램(메시지) 단위로 전송되며 그 크기는 65535바이트로, 크기가 초과하면 잘라서 보낸다.
    - 흐름제어가 없어서 패킷이 제대로 전송되었는지, 오류가 없는지 확인할 수 없다.
    - 파일 전송과 같은 신뢰성이 필요한 서비스보다 성능이 중요시 되는 경우에 사용된다.
   
- **정리**

| .          | TCP         | UDP         |
|------------|-------------|-------------|
| 연결 방식     | 연결형 서비스  | 비연결형 서비스 |
| 패킷 교환     | 가상 회선 방식 | 데이터그램 방식 |
| 전송 순서 보장 | 보장함       | 보장하지 않음   |
| 신뢰성       | 높음         | 낮음         |
| 전송 속도     | 느림        | 빠름          |

- **UDP의 패킷 손실 처리**
  - UDP의 손실된 패킷이나 순서에 관한 문제는 애플리케이션 레벨에서 해결
    - 1. 체크섬 : 네트워크를 통해 데이터를 전송하는 과정에서 패킷이 손상되었는지를 확인
    - 2. 재전송 요청(ARQ, Automatic Repeat reQuest) : TCP 재전송 메커니즘과 비슷하게, 애플리케이션 레벨에서 구현. 수신자가 패킷을 받으면 송신자에게 확인 메시지를 보내며, 패킷이 손상된 경우, 수신자는 송신자에게 패킷을 재전송하도록 요청
    - 3. 포워드 에러 수정(FEC, Forward Error Correction) : 해당 방식은 패킷이 손실될 경우, 이를 해결할 수 있는 패리티 데이터(보통 오류 정정코드)를 패킷에 추가해, 손실된 패킷을 복구하는 데 사용할 수 있도록 한다. 패킷의 크기가 증가해 네트워크 대역폭을 소비하지만, 패킷 손실에 대한 해결책을 제공하기에 고려해 볼 수 있는 사항 중 하나 이다.
  - 추가 체크섬에 관한 내용
    - https://whitehartlane.tistory.com/60
- 참조 :
  - https://inpa.tistory.com/entry/NW-🌐-아직도-모호한-TCP-UDP-개념-❓-쉽게-이해하자
  - https://mangkyu.tistory.com/15
  - https://f-lab.kr/insight/tcp-udp-differences-and-applications?gad_source=1&gad_campaignid=22368870602&gbraid=0AAAAACGgUFc-0uUMb_9LemFclao1RiA1T&gclid=CjwKCAjwjffHBhBuEiwAKMb8pENHYk8dTZ5jJQeqztqZbrQHX79jaXsOSbIsE1Yxl69E70MDgMU5choCe2sQAvD_BwE
  - https://velog.io/@pengoose_dev/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EC%99%80-%ED%8C%A8%ED%82%B7-c7ekavct
  - https://whitehartlane.tistory.com/60

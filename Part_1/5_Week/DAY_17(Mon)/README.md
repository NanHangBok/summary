# DAY_17

- [Spring 핵심모듈](#spring-핵심모듈)
- [프로젝트 구조](#프로젝트-구조)
- [Gradle](#gradle-빌드)
- [설정 파일](#설정-파일)
- [엔트리 포인트](#엔트리-포인트)
- [디버그](#디버그)

## SPRING 핵심모듈
seEL - 문법 관련 핵심 모듈
- spring-aop
  - AOP 관련 기능 제공 ( @Aspect, @Before, @After, Proxy 기반 interceptor 등 )
  - spring 자체의 트랜잭션, 보안 로직 구현 시 필수
  - *Proxy 기반 AOP (JDK Dynamic Proxy, CGLIB) 구현*
- spring-tx, spring-jdbc
  - spring-aop에 의존
  - 데이터 액세스와 트랜잭션 처리 기능 분리
  - spring-jdbc : jdbc 연동, jdbcTemplate, SQL 예외 처리
  - spring-tx : 선언적/프로그래밍 방식 트랜잭션 처리, @Transactional 지원
  - ORM 연동 (Hiberante, JPA) 시에도 spring-tx가 중간 조정자 역할
 
- http / 1. 무상태성 2. **무연결성**
- Spring Framework는 기능별 책임을 분리한 다수의 모듈로 구성된 계층형 프레임워크
- Spring 기반은 spring-core로부터 시작되며, spring-beans, spring-context를 거쳐 ApplicationContext 기반의 런타임 환경이 완성된다.
- spring-web이 HTTP 기반 기능을 담당하고, spring-webmvc가 MVC 구조를 완성하여 웹 애플리케이션 프레임워크로서의 SPRING을 형성한다.

---

## 프로젝트 구조
- src/main/java : 애플리케이션 소스 코드
  - 실행 가능한 애플리케이션 코드, 즉 비즈니스 로직의 중심이 되는 디렉토리
  - 패키지 구조는 GroupId와 ArtifactId로부터 자동 생성된 기본 패키지를 기준으로 구성된다.
  - @SpringBootAppication 어노테이션 - 실행 가능한 클래스
  - 프로젝트 루트 패키지 (Application.java)
  - controller(HTTP 요청을 처리하는 엔트리 포인트) (*Controller)
  - service (비즈니스 로직 처리 계층) (*Service)
  - repository (DB 접근 및 쿼리 처리 계층) (*Repository)
  - domain, entity (엔티티 및 핵 심 도메인 객체) (Member, Order, Product..)
  - 도메인 단위로 폴더를 나누는 방식 ( 계층형 vs 기능형 )도 존재
  - 컴포넌트 스캔 기준 - 현재 폴더 및 하위
- src/main/resources : 설정 파일과 정적 리소스
  - 리소스 파일을 저장하는 디렉토리
  - 애플리케이션의 **실행 환경 구성**과 **클라이언트 응답에 필요한 정적 파일**들이 위치
  - yml, properties
    - yml : 중복 코드를 줄 맞춤으로 줄일 수 있음
    - properties : 중복 코드를 전부 작성해야 함 / 가독성이 떨어짐
  - static/
    - html, img, css 등 서버에서 가공 없이 그대로 내려보내는 파일들
  - templates/
    - html을 렌더링할 때 사용하는 템플릿 파일들의 위치
    - Thymeleaf 템플릿 엔진 -> @Controller가 리턴한 문자열을 기반으로 해당 경로의 .html 파일을 찾아 렌더링
    - return "home" -> templates/home.html 렌더링
    - @RestController -> 리턴한 문자열을 그대로 return / .html 파일 없이 JSON파일 반환
  - banner.txt -> 애플리케이션 실행 시 콘솔에 출력되는 배너 커스터마이징 파일
  - messages.properties(messages.yml) - 다국어 지원(i18n)을 위한 메시지 파일
  - DB 비밀번호나 API 키, AWS와 같은 클라우드 접송 정보 등은 .gitignore 처리된 .env 또는 **외부 환경변수**(git secret)로 분리하는 것이 보안상 안전하다.
- src/test
  - 테스트 전용 소스 코드를 작성하는 영역
  - jar - 테스트ALL Pass -> 컴파일 -> 패키징 / **jar에는 테스트 코드를 포함하지 않음**
 
- 관습 기반 구성 (Convention over Configuration)
  - 설정보다 관습 원칙에 따르기 때문에 특정 위치에 파일을 두면 자동으로 인식하고 작동하도록 설계되어 있다.

---

## Gradle 빌드
소프트웨어 빌드를 자동화하는 도구 / 우리가 만든 코드를 실행 가능한 형태(jar,war 등)로 만드는 모든 과정을 자동으로 처리해주는 프로그램
- 자바 파일 컴파일 -> 라이브러리 다운,연결 -> 테스트 실행 -> 실행 가능한 애플리케이션을 만듦
- 애플리케이션을 실행 가능한 상태 직전까지 만드는 것
- 빌드
  - complie
  - Resolve Dependencies
  - Package : .class 리소스 등을 하나로 묶어서 jar, war 생성
  - Test
  - (옵션) Deploy : 빌드된 결과를 서버로 배포
- build.gradle : 빌드 설정과 의존성 정보를 담은 핵심 설정 파일
- gradle/ (디렉토리) : 래퍼 관련 구성 파일 포함

- Maven / Gradle
  - XML / Groovy,Kotlin
  - 문법 엄격 / 문법 유연하고 스크립트 스타일
  - 성능 느림 / 빠름
  - 읽기 비교적 명확 / DSL에 익숙해야 함
  - 확장성 낮음 / 높음

- build.gradle 구성
  - 플러그인 설정, 의존성 설정, 태스크 설정
  - 플러그인 설정
    - java / java 프로젝트로 인식하게 만듦
    - springframework.boot / spring-boot 관련 기능 제공
    - dependency-managerment / BOM설정 / BOM 방식으로 의존성 버전 관리 가능하게 함
   
  - 의존성 설정
    - dependencies{} 블록 안에 선언
    - 선언된 라이브러리는 Gradle이 자동으로 다운로드하고 프로젝트에 포함시킨다
    - implementation 실행 시 필요한 라이브러리 (컴파일+런타임)
    - testImplementation 테스트 시에만
    - compileOnly, runtimeOnly 컴파일/실행 시에만 필요한 경우
   
  - 태스크 설정
    - Gradle은 여러 태스크로 구성된 빌드 흐름을 갖는다.
    - 각 태스크는 빌드의 한 단계를 의미
    - 직접 만들 수도 있고 기존 태스크를 확장할 수도 있다.

---

## 설정 파일
- application.yml
  - spring.profile.active에 이름에 따라 application-*.yml이 실행 됨
  - 우선순위
    - 명령행 인자 ( -server.port=9090)
    - **application-{profile}.yml**
    - **application.yml**
    - 클래스 경로 외부의 설정 파일
    - @PropertySource 등 코드 기반 선언
    - 기본 값 (코드 내부에 선언된 값)

---

## 엔트리 포인트
- 메인 클래스의 역할
  - Spring boot 애플리케이션은 psvm 메서드를 진입점으로 실행되며 이 메인 클래스가 Spring boot 실행 구조의 **출발점**이자 **전체 설정 및 컴포넌트 구성**을 담당
  - psvm => 자바의 엔트리 포인트
- SpringApplication.run()
  - 스프링의 엔트리 포인트
  - ApplicationContext 생성, 설정 파일 로딩, 자동 구성 처리, 컴포넌트 스캔, 빈 생성 등
  - SpringApplication 객체 생성 및 초기화
  - ApplicationContext 생성 및 구성 -> 스프링 컨테이너 -> IoC 컨테이너
    - 구체적인 구현체는 WebApplicationType에 따라 동적으로 결정
    - Servlet 기반 -> 기본, HTTP 통신 / tomcat 서블릿 컨테이너
    - WebFlux 기반 -> 논 블록킹 처리 -> 병렬처리 / netty -> 서블릿 컨테이너 아님 / 리액티브
    - None
  - Auto Configuration 적용
    - @Conditional 계얼 어노테이션이 붙어 있어 조건이 맞는 경우 Bean으로 등록
  - Component Scanning 수행
    - @ComponentScan -> @Component, @Bean
  - ApplicationContext refresh
    - Bean 등록 및 초기화
    - CommandLineRunner, ApplicationRunner 실행
    - 전체 실행 흐름에서 다양한 시점에 이벤트를 발행하고 등록된 ApplicationListener 들이 이를 수신
  - 내장 서버 구동
    - Servlet 컨테이너 실행
    - WebServerFactory를 통해 DispatcherServlet을 Servlet으로 등록
   
- @SpringBootApplication
  - 단일 선언으로 자동 설정, 컴포넌트 스캔, 설정 클래스 등록을 포함하는 복합 구성
  - @ComponentScan
    - @Component, @Controller, @Service, @Repository 등이 선언된 클래스들을 자동으로 스프링 컨테이너에 등록
    - 수동설정 : basePackages, basePackagesClasses 속성을 사용하여 스캔 범위를 수동으로 설정 가능
      - @ComponentScan(basePackages = "com.example.demo.service")
      - @ComponentScan(basePackagesClasses - { MyService.class})
  - @EnableAutoConfiguration
    - Spring Boot의 핵심 자동 설정 기능을 활성화하는 어노테이션
    - 클래스패스에 존재하는 라이브러리, 설정 파일, 시존에 등록된 Bean 등을 기준으로 Spring 컨테이너에 필요한 설정을 자동으로 주입
  - @SpringBootConfiguration
    - 내부적으로 @Configuration과 동일한 역할을 수행
    - 구성 클래스로 사용될 클래스임을 명시하는 어노테이션
    - @Configuration은 내부적으로 @Component를 포함하고 있다
- DispatcherServlet , DataSource, MessageConverter, ErrorController >> 필수 Bean
- 후처리기 적용
  - Bean이 생성된 후, 또는 초기화되기 직전/후에 Spring이 개입하여 추가 동작을 수행할 수 있도록 하는 기능
  - Bean이 등록되는 시점은 **정확한 생명주기**와 **우선순위**를 가짐
 
#### 이벤트 리스너와 생명주기 콜백 ( 트리거 )
- Spring Boot는 실행 도중 다양한 생명주기 이벤트를 발생시키며 이를 통해 개발자가 특정 동작 삽입 가능
- 콜백 매커니즘
  - @PostConstruct : 빈 생성 후 자동 호출
 
## 디버그
- 개발 도중 예상과 다른 결과가 나타나는 원인을 파악
- 변수 값, 메서드 호출 순서, 조건문 분기 등을 확인
- 버그나 잘못된 흐름을 수정할 수 있도록 돕는다

#### 디버거
- 개발자가 원하는 지점에서 멈추고 현재 상태를 관찰할 수 있도록 도와주는 도구
- 중단점(Breakpoint) 지정한 코드 지점에서 실행 일시 정지
- Resume Program 브레이크포인트를 넘기고 다음 브레이크포인트까지 실행
- step Into 메서드 내부로 진입
- Step Over 메서드 실행은 건너뛰고 다음줄로 이동
- Step Out 현재 메서드 종료 후 상위 호출 지점으로 복귀
- Variable Watch 변수 값 실시간 확인
- Call Stack 확인 / 현재까지의 메서드 호출 순서 추적 가능

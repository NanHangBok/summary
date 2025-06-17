# DAY_18

- [브레이크 포인트](#브레이크포인트-설정)
- [Spring boot Starter 의존성](#starter-의존성)
- [Spring boot Starter](#starter)
- [의존성 관리](#의존성-관리)
- [SpringBoot 계층형 아키텍처](#spring-boot-계층형-아키텍처)
- [이벤트 기반 아키텍처](#이벤트-기반-아키텍처)
- [Bean](#bean)
- [의존관계](#의존관계)

## 브레이크포인트 설정
- main()의 run() 호출 / 애플리케이션 시작
- SpringApplication() 생성자 / 실행 타입, 환경 설정 초기화
- prepareContext() / 컨텍스트 구성, 리스터 초기화
- refresh() / Bean 등록과 생명주기 처리
- callRunner() / 초기 비즈니스 실행 시점
  - 톰캣 대기 상태

## starter 의존성
- 소프트웨어 의존성
  - 하나의 모듈 또는 프로그램이 동작하기 위해 다른 모듈이나 라이브러리 의 기능을 참조하거나 사용하는 관계
  - 하나의 모듈 또는 프로그램이 다른 모듈이나 라이브러리의 기능 없이는 동작할 수 없는 상황
- Spring boot 의존성
  - Maven 또는 Gradle과 같은 빌드 도구에 기반하여 의존성을 선언하고 관리
    - 의존성 자동 다운로드
    - 하위 의존성 자동 해결
    - 버전 충돌 해결
    - 보안 업데이트 및 호환성 자동 반영
  - starter -> 자주 사용되는 의존성 조합 패키징
 
## starter
- 특정 기능을 구현하는 데 필요한 의존성 묶음
- 해당 starter와 관련된 라이브러리들과 기본 설정이 함께 포함된다.
- 개발자는 어떤 라이브러리를 골라야 할지, 버전 충돌은 없는지 등을 고민하지 않고, Spring Boot 팀이 검증한 조합을 그대로 사용할 수 있다.
- ./gradlew dependencies 의존성 트리 확인

## 의존성 관리
- Spring boot : BOM 관리
  - BOM은 의존성들의 버전 정보를한 곳에 모아 관리하는 POM 파일이다.
  - Spring boot에서는 spring-boot-dependencies 라는 BOM을 제공
  - 개발자는 Starter만 선언하면 해당 BOM을 통해 정해진 라이브러리들이 자동으로 매핑된다.
  - 장점
    - 일관된 버전 구성
    - 자동 버전 지정
    - 업그레이드 용이성
    - 보안 패치 반영 용이
   
#### spring-boot-dependencies
- 모든 Starter들은 공통적으로 spring-boot-dependencies BOM에 의존
- BOM
  - Spring 프레임워크 주요 모듈
  - 서드파티 라이브러리
  - 테스트 프레임워크
 
#### 의존성 제외와 대체
- 특정 하위 라이브러리 제외 ( Gradle -> exclude )
  - exclude group: '~~', module: '--'
  - exclude group 키워드와 제거할 module을 명시하여 의존성 트리에서 제거
  - 새로 implementation을 통해 대체할 라이브러리로 교체 가능
- 한계
  - 의존성을 한 번 제외하더라도 의도치 않게 다른 라이브러리가 동일한 의존성을 다시 참조할 수 있음
  - 삭제되지 않고 재설치가 이루어짐 -> 반복적으로 exclude 선언 필요
  - configuration.all { exclude group: '--', module: '--' } 가능 / 전체 삭제
- 주의 사항
  - 다른 모듈과의 의존 관계를 따져보지 않고 제외할 경우 ClassNotFoundException 발생 가능

#### 선택적 의존성
- 스코프를 활용한 분리
  - implementation 일반 의존성
  - developmentOnly 개발 환경 전용
  - compileOnly 컴파일 시만 필요, 런타임 미포함 / Lombok
  - runtimeOnly 런타임에만 필요 / DB 관련
  - testImplementation 테스트 전용
  - developmentOnly는 spring-boot에서 지원하는 기능
 
#### 버전 오버라이딩
- 의도적으로 특정 라이브러리의 버전을 변경(오버라이딩) 해야 하는 경우
  - 보안 이슈가 있는 구버전에서 벗어나야 할 때
  - 최신 기능을 테스트하고 싶을 때
  - 특정 하위 의존성에서 충돌이 발생할 때
  - implementation '--:--:version(2.1.2)'
- dependencyManagement 직접 사용
  - dependencyManagement { import { mavenBom "--:--:--"}}
- 리스크
  - 의도하지 않은 비호환 발생
  - NoSuchMethodError
  - ClassCastException
  - ConfigurationProperties 매핑 실패
 
## spring boot 계충형 아키텍처
- 계층별 구성
  - Client / HTTP request를 주고 Controller를 통해 response를 받는다
  - @Controller / request, response / 요청 수신, 응답 반환 / 클라이언트의 요청을 받아 처리 흐름을 제어
  - @Service / 비즈니스 로직 / 핵심 도메인 로직과 트랜잭션 처리 담당
  - @Repository / DAO / 데이터 접근 (DB와 직접적인 통신 수행)
  - 생성자 자동 주입 - 빈의 타입 -> 같으면 빈의 이름 -> 모두 같으면 터트림

- Controller / @RestController
  - RestController -> @ResponseBody를 포함한다 / ***JSON타입으로 반환한다***

- Service
  - **비즈니스 로직 처리 담당**
  - 트랜잭션 처리, 도메인 규칙 구현 등 핵심 작업을 수행
  - @Transactional과 함께 자주 사용 됨
 
- Repository
  - **DB와 직접 통신하는 DAO(Data Access Object) 역할**
  - 내부적으로 @Component + 예외 변환 기능 포함 (Spring DataAccessException 변환)
 
#### 계층 간 의존성 관계
***계층 간의 일반향 의존성***
- 일반적 흐름
  - 관심사의 분리
  - Controller -> Service -> Repository
  - Controller는 Service 호출, Service는 Repository 호출
  - Controller는 Service에 의존, Service는 Repository에 의존
 
- 같은 계층끼리의 의존
  - Service가 다른 Service를 참조할 수 있다
  - 자신의 비즈니스 로직을 처리하는데 **다른 서비스의 기능이 논리적으로 필요한 경우**
  - **일방향(단방향)** 이어야 함
  - **역할과 책임이 명확해야 함** / 유지보수 UP
  - 계층적 위계가 어긋나지 않아야 함
  - 순환참조 시 협력 로직을 별도의 조정자로 위임
    - OrderService, MemberService를 참조하는 별도의 OrderProcessManager 클래스 생성
    - 서비스가 서로 직접 물려있지 않고 조정자(중간자)를 거쳐 연결


## 이벤트 기반 아키텍처
**시스템의 흐름을 이벤트라는 신호나 메시지를 중심으로 처리하는 방식**<br>
*어떤 동작이 발생했을 때 발생시키는 신호*, 해당 이벤트를 처리하는 *리스너*가 이를 감지하고 필요한 작업 수행<br>
"누가 발생시켰는지 알 필요 없이, 무언가 일어났다는 사실에 반응하는 방식"

- 핵심 개념
  - 이벤트 : 시스템에서 발생한 사실 또는 행위
  - 이벤트 발행(Publish) : 이벤트 객체를 생성하고 시스템에 전파
  - 리스너 : 특정 이벤트가 발생했을 때 동작을 정의하는 컴포넌트
  - 비동기 처리 : 이벤트와 리스너는 일반적으로 느슨하게 결합되며, 필요에 따라 비동기로 작동

- 흐름
  - 어떤 일이 발생 (회원가입)
  - 이벤트 객체 생성 (UserRegisteredEvent)
  - 이벤트 발행 (EventPublisher)
  - 이벤트 리스너가 이 이벤트를 감지하여 추가 작업 수행 (이메일 발송)
 
- 장점
  - 결합도를 낮춤 / 이벤트 발행하는 쪽과 처리하는 쪽이 서로 직접 참조하지 않아도 됨
  - 관심사 분리 / 주요 비즈니스 로직과 부가 처리를 명확히 분리할 수 있음
  - 확장성 향상 / 이벤트 리스너만 추가하면 기능을 유연하게 확장 가능

- 모든 이벤트가 비동기 방식은 아님 **Default는 동기**
- 이벤트의 흐름이 명시적이지 않아 코드 추적이 어려움 / 많이 사용하면 찾기 어려움

#### 이벤트 처리 구조
- 객체 정의
  - 일반적인 POJO클래스 형태로 정의 (생성자, 게터/세터)
  - 일반적으로 불변을 유지 ( Setter 가 없음 )
- 이벤트 발행
  - ApplicationEventPublisher 인터페이스를 통해 이벤트 발행
  - publishEvent()로 컨텍스트에 이벤트를 전파 (ApplicationEventPublisher.publishEvent())
  - 해당 이벤트를 수신하는 모든 리스너 호출
- 이벤트 수신
  - @Component 로 빈으로 등록된 클래스
  - @EventListener 특정 이벤트 타입을 처리할 메서드 정의
  - @EventListener는 특정 이벤트 타입을 감지하여 자동으로 메서드를 실행해주는 어노테이션
  - 비동기 처리 : **@Async**가 붙은 리스너는 비동기 처리가 됨
    - *@SpringBootApplication 어노테이션*이 있는 메인 클래스에서 **@EnableAsync 어노테이션** 추가
  - @Order 어노테이션으로 리스너의 실행 순서 조정 가능


## Bean
**Spring IoC 컨테이너에 의해 관리되는 객체** <br>
Spring이 그 생명주기와 의존성 주입을 책임진다 <br>
객체의 생성, 초기화, 사용, 소멸까지 **전 과정**을 개발자가 아닌 **컨테이너가 제어하는 구조**

- POJO 와 Bean
  - 비침습성 : 객체 자체는 Spring 프레임워크에 의존하지 않는다
  - 기존 코드를 재사용 가능 : 테스트, 유지보수, DI 적용 용이

- 메타정보
  - Bean 이름 (ID) : 고유 식별자
  - 클래스 정보 : 인스턴스화할 대상 클래스
  - 의존성 정보 : 주입되어야 할 다른 Bean 목록
  - 스코프 정보 : singleton, prototype 등 / Default는 singleton
  - 초기화/소멸 메서드 : initMethod, destroyMethod 등 / singleton의 소멸시점 : 프로그램(어플리케이션) 종료 시
  - 생성시점 : Lazy or Eager / Default는 Eager
 
- Bean 정의 충돌이나 중복 등록 오류는 피해야 한다
 
- Spring IoC 컨테이너에서의 Bean 생명주기
  - 생성 : 기본 생성자 또는 @Bean 어노테이션이 정의된 메서 등을 통해 인스턴스 화
  - 의존성 주입 : 생성자, 필드, Setter 등을 통해 필요한 Bean 주입
    - 의존관계가 더 적은 것부터 생성 / Why? 생성자로 주입해야할 Bean이 초기화 되어 있어야 주입이 가능하니까
    - 생성자 주입으로 객체가 불변임 -> null이 아님을 보장
  - 초기화
  - 사용 : 비즈니스 로직 수행
  - 소멸

#### IoC 컨테이너
- Bean 생성, 의존성 주입, 생명주기 관리를 하는 중앙관리 장치
- 2가지 핵심 인터페이스가 BeanFactory, ApplicationContext이다.

#### BeanFactory
- Spring 컨테이너의 최상위 인터페이스, Bean의 생성 및 조회에 필요한 최소 기능만 제공
- Lazy Initialization (지연 초기화)
- getBean() 메서드를 통한 명시적 Bean 조회 중심 / 사용안함 IoC, DI, DIP 위반

#### ApplicationContext
- BeanFactory를 상속한 일종의 확장 컨테이너 인터페이스
- 메시지 소스 처리 (i18n) / 메시지 관련 인터페이스 상속
- 이벤트 발행 및 수신 / 이벤트 관련 인터페이스 상속
- AOP 연동
- 빈 후처리기 / 빈 관련 인터페이스 상속 (ListableBeanFactory) -> BeanFactory 상속
- 자동 Bean 등록 지원

- AnnotationConfigApplicationContext / @Configuration 클래스 기반

#### java 기반 bean 설정
- @Configuration + @Bean 사용
- @Configuration 클래스
  - 해당 어노테이션을 특정 클래스 레벨에 정의하면 해당 클래스는 구성 클래스가 된다
  - 해당 클래스 내부에는 **하나 이상의 @Bean 메서드를 정의** -> 반환 객체는 Bean으로 등록 됨
  - 내부적으로 @Component가 포함 -> 컴포넌트 스캔의 대상이 된다
 
- @Bean 메서드
  - 메서드 단위에 선언
  - 반환하는 객체를 Spring Bean으로 등록
  - 반환 타입은 일반 POJO 클래스이며 제약은 없음
  - 의존성이 있는 다른 Bean 메서드를 호출함으로써, 명시적 DI 구성도 가능
  - @Bean 메서드 명이 Bean의 이름으로 등록 된다
  - bean의 이름은 중복될 수 없으며, 동일 이름이 있을 경우 덮어쓰기 또는 예외가 발생
  - 명시적 이름을 부여할 수도 있다 @Bean(name="asdf")

#### 컴포넌트 스캔
@Component 어노테이션을 통해 특정 패키지 이하의 클래스들을 **자동으로 탐색하여 Bean으로 등록**할 수 있다
- 기본 탐색 시작점은 *해당 어노테이션이 정의된 클래스가 위치한 패키지*이다.
- @Component, @Controller, @Service, @Repository, @Configuration 등 등록

- basePackages 옵션을 사용하여 탐색 경로를 수동으로 지정할 수 있으며, *다수의 패키지 경로*를 지정할 수도 있다.
- basePackagesClasses 옵션을 통해 *클래스 기반*으로 탐색 범위를 정할 수 있다. / 리팩토링 시 더 안전한 방식

- includeFilters, excludeFilters 옵션을 통해 포함/제외 대상을 필터링할 수 있다.
- FilterType
  - ANNOTATION 특정 어노테이션이 붙은 클래스만 포함/제외
  - ASSIGNABLE_TPYE 특정 클래스 또는 인터페이스 구현체 포함/제외
  - REGEX 정규 표현식 기반으로 클래스 이름 필터링 / 패턴 찾기 / 주민등록번호(??????-???????), 전화번호(???-????-????) 등
- @ComponentScan(basePackages="com.~", excludeFilters = { @ComponentScan.Filter(type=FilterType.ANNOTATION, classes={org.~~.~.~.class})})

#### SpringBootApplication
Spring Boot 애플리케이션의 진입점에 붙는 핵심 어노테이션
- @SpringBootConfiguration : 설정 클래스임을 명시. 내부적으로 @Configuration 포함 -> bean으로 등록, 관리
- @EnableAutoConfigurtaion : 클래스패스, 설정 파일, 조건 등을 기반으로 Bean 자동 등록 수행 -> 스프링 내부에서 사용하는 Bean
- @ComponentScan : 현재 패키지 기준 하위 클래스의 @Component 자동 탐색 -> 개발자가 만든 Bean

- 자동 등록된 Bean 덮어쓰기 ( @EnableAutoConfiguration을 통해 만들어진 Bean 덮어쓰기 )

- 자동 등록된 Bean
  - WebMvcAutoConfiguration
  - DataSourceAutoConfiguration
  - JacksonAutoConfiguration
  - SecurityAutoConfiguration ...
 
- @Bean 정의를 통한 덮어쓰기
  - 같은 이름이나 타입의 Bean을 직접 정의하면, Spring Boot의 자동 구성 Bean을 대체할 수 있다
- 설정 속성을 통한 Bean 커스터마이징
  - .yml, .properties를 수정해서 구조를 변경
  - *지원하는 부분 설정만 변경 가능*

- 자동 등록 제외시키기
  - 자동 구성되는 특정 기능이 불필요하거나 충돌을 유발할 때 제외
  - @EnableAutoConfiguration(exclude = ...) 활용
  - @SpringBootApplication(exclude = {SecurityAutoConfiguration.class})
 
  - application.yaml을 통한 제외
    - 프로퍼티 기반으로 비활성화 가능
    - exclude에는 전체 클래스 이름(FQCN)을 명시해야 한다.
    - ex: org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration

## 의존관계
#### 의존성 주입 방법
*IoC 원칙에 따라 의존성 주입(DI)을 제공* -> **결합도 낮추고 유연한 설계** (생산성 향상)
- 생성자 주입
  - 생성자가 1개일 때 @Autowired 생략 가능
  - effective final ( 불변 / null이 아님을 보장 )
  - 객체 간 의존성 보상 ( 순환참조를 spring 실행 시점에 에러를 발생시킴 (런타임에러 방지) )
- setter 주입
  - @Autowired 생략 불가 -> 생략하는 상황은 생성자로 초기화하고 생성자가 1개일 때 가능
  - 주로 의존성이 선택적이거나, 변경이 가능한 경우 사용되는 방식
- 필드 주입
  - @Autowired 생략 불가
  - 리플렉션 없이 목 객체 주입 어려움
  - 불투명한 의존성, DI 프레임워크에 강하게 의존, 테스트 어려움 -> 사용하지 않음

#### 동일 타입 Bean 처리
- @Primary
  - 여러 개의 동일 타입 Bean 중에서 기본값으로 주입할 대상을 지정하는 방식
  - 해당 어노테이션이 붙은 Bean 우선 주입
  - @Qualifier보다 우선순위가 낮음
  - 공통 구현체에 @Primary를 붙여두고, 테스트나 특정 상황에서는 @Qualifier로 다른 구현체를 주입할 수도 있다.
- @Qualifier
  - 정확한 Bean 이름을 지정(명시적 지정)함으로써 모호성 제거
  - 생성자, 필드, 메서드 등 어떤 위치에서도 사용 가능 ( 클래스명 사용 / 시작은 소문자로 변경 )

# DAY_19

- [Bean](#bean)
  - [동일한 bean처리](#동일-타입-bean-처리)
  - [다양한 의존관계](#의존-관계)
  - [스코프(Scope)](#스코프)
  - [라이프 사이클](#life-cycle-콜백)
- [설정 정보 외부화](#설정-정보-외부화)
  - [@Value](#@value)
  - [@ConfigurationProperties](#@configurationproperties)
  - [Environment](#environment)
- [@Profile](#@profile)
- [@Conditional](#@conditional)
- [Bean 순서 등록 제어](#bean-순서-등록-제어)

## Bean
#### 동일 타입 Bean 처리
- 명시적 선택 ( @Qualifier )
  - 기본 이름은 클래스명 ( 맨 앞글자 소문자 )
- 빈 이름을 통한 식별
  - @Resource
    - Jakarta EE 표준 어노테이션으로 JDK 9 이후 기본 JDK 모듈에서 제거 -> 이후부터는 별도 의존성 추가 필요
    - Spring Framework에서 제공하는 어노테이션이 아니라 Java 기본 어노테이션이라 생성자 주입 불가
    - 필드 주입과 세터 주입만 가능
    - @Resource(name ="inMemoryRepository"), @Resource(type="MemberRepository.class") 가능=

  - @Inject
    - Java에서 제공하는 기본 어노테이션
    - Type으로 빈을 의존성 주입함.
    - 생성자 주입, 필드 주입, 생성자 주입 모두 가능
    - @Inject @Named("...") or Service(@Named("...") Repository repository) 가능

#### 의존 관계
- 선택적 의존 관계 / 값이 있을수도 null일 수도...
  - @Autowired(required = false)
    - 생성자 주입 시 해당 어노테이션 사용 불가
    - 해당 Bean이 존재하지 않아도 주입 과정에서 예외를 발생시키지 않음
  - @Nullable
    - 못 찾으면 null 주입 -> 컴파일 단계에서 Framework가 예외 처리를 발생시키지 않음
  - Optional<T>
    - 타입을 Optional<T> 로 설정 가능 -> null을 받을 수 있음
    - null을 포함한 모든 객체를 Optional로 래핑중

- 컬렉션/배열 의존관계
  - 동일한 타입의 Bean이 여러개 존재할 경우 List, Map, Set, array 등 자동 주입 가능

- 순환참조
  - 2개 이상의 Bean이 서소를 주입하려고 할 경우, 순환 참조 발생 -> Framework가 사전에 막음
  - application.yaml을 수정해서 허용 가능. 단, 설계 리팩토링이 올바른 해결책
  - -> 중간자 클래스를 생성해서 역할 위임

#### 스코프
**생성된 Bean의 생명 범위**
- Singleton
  - Spring의 기본 스코프
  - 컨테이너 내에 Bean 객체가 하나만 존재
  - 컨테이너가 시작될 대 객체가 생성되고, 모든 의존 주입 지점에서 같은 인스턴스를 공유
  - 상태를 내부에 유지하면 동시성 문제가 발생할 수 있다. / 공유되면 안되는 필드를 가지고 있을때 문제 발생
 
- Prototype
  - 주입될 때 마다 새로운 인스턴스 생성
  - Bean을 생성한 후 Spring은 생명주기 관리를 즉시 종료, 소멸 메서드 호출 안함 (생성, 주입, 초기화) / 개발자가 소멸 시켜야 함
  - 자주 상태가 바뀌거나, 무거운 작업을 매번 다른 객체로 수행해야 할 때 유용

- 웹 스코프
  - 웹 환경에 특화된 Bean 생명주기 관리를 위해 제공
  - WebAppliationContext에서만 유효
  - 일반적으로 컨트롤러, 서비스, 필터 내부에서만 사용 가능 -> @WebMvcTest 등 별도의 설정 필요
  - request
    - HTTP 요청 , 요청 시 생성, 요청이 끝나면 소멸 / 사용자 요청 처리용 Bean
  - session
    - 사용자의 정보를 저장, 서버에 보관, 안전하다
    - 사용자의 세션 단위로 Bean이 유지
    - HTTP 세션, 세션 시작 시 생성, / 로그인 상태 유지, 장바구니 등에 사용
  - application
    - ServletContext 전체, 애플리케이션 시작 시 생성, 애플리케이션 전역에서 사용 / 전체 방문자 수 등 전역 통계에 사용
    - Singleton과 유사 웹 전용 Singleton
    - 단일 WAS(Web Application Service,Server) 컨텍스트 (Servlet 컨테이너) 내에서 모든 요청/세션이 공유
    - WAS -> Servlet 컨테이너, Tomcat
    - WAS 단위의 공용 저장소에 가까움

WAS/ Web Server
- WAS : 여러개의 작업을 진행함 Web Server 이상의 일을 함
- Web Server : 정적 데이터 반환 ( 이미지, 오디오, 비디오 등... )
- 둘 다 HTML을 반환하지만 WAS는 동적 HTML(사용자에 따라 화면이 다름), Web Server는 정적 HTML(모든 사용자가 동일한 화면을 봄)

- 스코프 프록시(proxyMode)
  - 웹 스코프를 사용하는 Bean은 싱글톤 Bean(예:서비스나 컴포넌트)에 직접 주입하면 동작 오류가 발생할 수 있음
  - 주입 시점에는 웹 스코프 객체가 존재하지 않기 때문에 -> 제공되는 스코프 프록시 기능으로 해결
  - 객체의 대리자 역할을 수행하는 객체 / 원래 객체를 감싸서 기능을 확장하거나 호출 흐름을 제어할 수 있도록 함
    - 지연 초기화, 요청/세션 등 스코프 별 객체 관리 / 트랜잭션 관리, 로깅, 보안 등 AOP 적용 등에 활용
  - 프록시 종류
    - ScopedProxyMode.TARGET_CLASS, INTERFACES
    - CGLIB 기반 (default)
      - 대상 클래스 자체를 상속하여 프록시르 생성
      - 인터페이스가 없는 클래스, 더 높은 성능과 유연성(권장)
      - 바이트코드를 조작하여 프록시 클래스를 생성

    - JDK 동적 프록시
      - 대상 객체가 구현한 인터페이스를 기반으로 프록시 생성
      - 클래스에 인터페이스가 있는 경우
      - INTERFACES, TARGET_CLASS 둘다 가능
      - 리플렉션 기반으로 인터페이스 메서드를 가로채고, 실제 타켓 객체에게 위임

#### life-cycle 콜백
- 특정 타이밍에 작업이 필요한 경우 자동으로 제어할 수 있도록 제공하는 메서드
  - initMethod(Bean 생성 직후), DestroyMethod(컨테이너 종료 시 호출, 싱글톤 스코프에서만 동작 / 소멸을 개발자가 컨트롤, 초기화 이후에는 컨테이너가 관리하지 않음)
 
  - @PostConstruct(Bean 생성 후 호출/의존성 주입 완료 시점), @PreDestory(ApplicationContext 종료되기 전 호출)

  - InitializingBean, DisposableBean
    - 라이프사이클 인터페이스
    - afterPropertiesSet() 초기화 로직 / 의존성 주입 완료 후 호출
    - destory() 소멸 로직 / 컨테이너 종료 시 호출
    - 인터페이스에 강하게 결합됨
   
- 순서
  - @PostConstruct, afterPropertiesSet(), init-method
  - @PreDestroy, destroy(), destroy-method 순서

## 설정 정보 외부화
#### 필요성
데이터베이스 정보, 포트 번호, API키 등 이런 값들을 코드 내부에 직접 작성하는 것은 **보안성, 유지보수성, 확장성 측면에서 여러 문제**를 초래할 수 있다.<br>
Spring은 설정 정보를 .properties, .yaml, 환경변수(.env)등 다양한 방식으로 외부화 지원
- 환경별 설정 분리
  - 개발(Dev), 테스트(test), 운영(prod) 환경은 서로 다른 설정이 필요
  - 동일한 설정을 모든 환경에서 사용하면 예기치 못한 문제나 보안 사고가 발생할 수 있다.
  - Spring Boot는 application-{profile}.yaml 같은 환경별 프로파일 구성을 통해 이 문제를 해결한다.
 
- 민감 정보 관리
  - API Key, DB 비밀번호 같은 민감 정보는 하드 코딩 시 노출 위험
  - 외부 설정 파일 또는 환경 변수 등을 통해 민감 정보를 분리함으로써 보안 수준을 높일 수 있음
  - Spring Boot는 환경 변수, 시스템 프로퍼티, 암호화된 설정 파일 등 다양한 방식의 외부 주입을 지원 (os 환경 변수) / ${PASSWORD}

- 운영 환경에서의 설정 변경
  - 코드 수정 없이 설정만 바꿔도 운영 환경을 변경할 수 있어야 함
  - Spring Boot는 설정 정보를 런타임 시점에 반영하도록 다양한 확장 방식 제공
 
#### @Value
Spring에서 가장 기본적인 속성(Property) 주입 방법<br>
application.yaml, application.properties에 정의된 설정 값을 가져와 Bean 필드에 주입 할 수 있음
- 프로퍼티 주입
  - my : greeting : "hello" -> @Value("${my.greeting}")private String greeting;
  - > sout(greeting); // hello

- spEL 활용
  - 단순 값 외에도 SpEL 지원, 동적 계산이나 조합도 가능
  - @Value("#{${my.greeting}} 하고 인사합니다")
  - @Value("${my.greeting} 인사합니다") / #{} 생략 가능
  - @Value("#{${my.number} * 2}") / 연산도 가능

- 기본값 설정
  - 설정 값이 존재하지 않을 경우 기본값 지정 가능
  - @Value("${my.username:anonymous}")
  - username 설정이 없으면 "anonymous" 자동 지정
  - Boolean, Integer와 같은 기본형에도 활용 가능

#### @ConfigurationProperties
계층적이고 복잡한 설정 정보를 타입 안전하게 주입할 수 있는 방법
- application.yaml
  - my : service : name : "~" port : 1234 enabled : true
  - @Component @ConfigurationProperties(prefix = "my.service")
    - private String name; private int port; private boolean enabled;
- 계층형 가능
  ```
  my :
    greeting: "hi"
    user :
      id : 1
      port : 1234
      name : "adsf"
  ```
  ```java
  @Component
  @ConfigurationProperties(prefix = 'my')
  @Getter
  @Setter
  public class My {
    private String greeting;

    @Getter
    @Setter
    public static class User {
      private int id;
      private int port;
      private String name;
    }
  }
  // 이런식으로 가능하다
  ```

- 리스트, 맵 바인딩
  ```
  my :
    users :
      - name : "abc"
        age : 10
      - name : "aaa"
        age : 12
      - name : "def"
        age : 15
  ```
  - 리스트 형식
    - private List<User> users;
   
    - public static class User {
      - private String name;
      - private int age;
  - Map 형식
    - private Map<String,String> users;

- 유효성 검사
  - 의존성 spring-boot-starter-validation 추가 필요
  - @Validated 유효성 검사를 하기 위한 클래스에 적용
  - @NotNull, @Min, @Range, @NotBlank ... 유효성 검사를 할 필드에 적용
  - 애플리케이션 구동 시점에 에러를 발생시킴
  - 커스텀 어노테이션 생성 가능

#### Environment
Spring Boot는 설정 정보를 수집하고 이를 PropertySource로 관리
- application.yaml
- application.properties
- 명령줄 인수(--server.port=8081)
- 환경 변수 (SERVER_PORT)
- JVM 시스템 속성 (-Dserver.port=8082)
여러 출처에서 온 설정값(키-값)들을 모두 통합하여 Environment가 관리하게 되는 구조

- 우선순위
  - 명령줄 인수
  - OS 환경 변수
  - JVM 시스템 속성
  - application-{profile}.yaml
  - application.yaml or application.properties
  - @PropertySource or @TestPropertySource로 명시한 외부 파일
  - Spring Boot 기본값

- 프로퍼티 소스 추가 가능

- Environment 객체 직접 활용 가능
  - 주입 받아 직접 값을 읽는 것 가능

## @Profile
환경별로 서로 다른 Bean을 등록하거나 설정을 적용할 수 있도록 도와주는 기능
- @Profile("dev") -> 현재 활성화된 프로필이 dev일 때에만 해당 Bean이 등록
- @Profile({"dev", "test"}) 둘 중 하나라도 활성화 되어 있으면 Bean 등록 (or)

- 프로필 활성화 방법
  - application.yaml or application.properties 에 spring: profiles: active: dev 설정
  - VM 런타임 설정 -> 프로필 입력 (dev)
  - 커맨드라인에서 실행 시 지정 (./gradlew bootRun --args='--spring.profiles.active=dev')
 
- 다중 프로필 활성화
  - application.yaml = spring: profiles: active: - dev - prod
    - 활성화 시킬 프로필을 배열로 제공
    - main : allow-bean-definition-overriding : true 로 Bean 오버라이딩 허용
    - 조건부 Bean들 중 하나라도 매칭되면 활성화
    - Bean 이름이 겹치면 충돌남 -> 다른 이름으로 명시 (@Bean("aInfoPrinter") @Bean("bInfoPrinter))


## @Conditional
**특정 조건이 충족될 때만 특정 Bean을 등록하도록 하는 기능** <br>
*개발자가 직접 조건 클래스를 만들어 사용할 수도 있음*
- application.yaml
  - custom: feature-x: true
  - -> custom.feature-x 값이 true 일 때만 특정 Bean을 등록하도록 설정
- FeatureXEnabledCondition 클래스 -> Condition 인터페이스를 구현하여 사용자 조건 정의
  - Environment에서 설정값을 조회하여 조건을 판별
- 조건부 Bean 클래스 FeatureService
- 외부에서 사용하는 객체(Bean 등록 안함) -> @Configuration -> FeatureConfig
  - @Conditional(FeatureXEnabledCondition.class)
  - 조건이 true면 Bean 생성
 
- @ConditionalOnProperty
- @ConditionalOnMissingBean
- @ConditionalOnBean
- @ConditionalOnClass / @ConditionalOnMissingClass
- @ConditionalOnExpression SpEL기반 ...

## Bean 순서 등록 제어
어떤 Bean을 먼저 생성하고, 어떤 순서로 의존성을 주입하고, 어떤 순서로 초기화하는지 의미<br>

- 기본 원칙 : 의존관계에 따른 자동 순서
  - 작은거 먼저 생성 ( 의존관계가 적은 순서 )
  - 보통 의존성 주입에 따라 적절히 Bean을 초기화 하므로 순서 제어가 필요하지 않음

- 순서 강제의 필요성
  - 직접적인 의존 관계가 없는 Bean 간 순서 지정
    - 데이터 소스 초기화 -> 캐시 초기화 -> 서비스 실행 순서가 필요할 때
  - 외부 시스템 연결을 선행해야 하는 경우
    - Redis 연결이 완료된 이후, Redis를 사용하는 서비스 Bean이 초기화되어야 할 때
  - 초기화 로직의 선후 관계가 중요한 경우
    - 공통 설정을 먼저 적용한 뒤 각 서비스가 초기화되도록 해야할 때

  - -> **DI 그래프만으로는 유추할 수 없는 실행 순서 의존성이 있는 경우**
 
- 잘못된 순서로 인한 문제
  - *BeanNotFoundException*
  - *NullPointerException*
  - 초기화 실패 및 장애
  - 불안정한 테스트 결과

#### 순서 제어
- @DependsOn
  - 해당 Bean이 초기화되기 전에 반드시 먼저 초기화되어야 하는 Bean 이름을 지정하는 어노테이션 ( 직접 포함 시 / 결합도가 높을 때 )
  - -> 어노테이션이 **붙은 클래스가 초기화되기 전**에 어노테이션에 **지정된 클래스가 초기화 되어야 함**
  - **DB 연결 시 사용 됨**
  - 주의 사항
    - 의존성 주입이 아닌 **순서 제어만**
    - 잘못된 이름 명시하면 무시 됨
    - 너무 많은 Bean 사이에 @DependsOn을 사용하면 전체 Bean 초기화 순서를 복잡하게 만들 수 있음. >> 가능한 최소화으로

- @Order
  - Bean들이 순서대로 처리되어야 할 때, 실행 순서를 지정하는 어노테이션
  - 동일 타입 Bean이 여러 개 있을 때, 그 **처리 순서** 지정
  - **번호가 낮은 순서부터 실행**
  - 인터페이스 기반 초기화/실행 처리 시 우선순위를 설정
  - @Component에 적용
  - CommandLineRunner, ApplicationRunner
  - BeanPostProcessor, ApplicationListener
  - 주의 사항
    - 기본값은 Ordered.LOWEST_PRECEDENCE = Integer.MAX_VALUE
    - 가장 높은 우선순위 = Order(0) or  Ordered.HIGHEST_PRECENDENCE
    - Bean 간의 명시적 의존 관계를 설정하지 않음 / **순서만 지정**하고 싶을 때 사용
    - 빈 등록 순서가 중요하지만 직접 의존하지 않을 때 적합 / Bean 간 강한 연관이 있다면 @DependsOn 이나 생성자 주입으로 처리

- Ordered 인터페이스
Spring Framework가 Bean 또는 콜백 인터페이스를 순서대로 처리할 수 있도록 제공하는 우선순위 인터페이스
  - 인터페이스 구현 후 *getOrder() 메서드 오버라이드*
  - 외부에 다른 인터페이스를 상속받아 사용할 때 같이 상속받아서 사용
  - ex: implementation BeanPostProcessor, Ordered {}
    ```java
    public class FirstProcssor implementation BeanPostProcessor, Ordered {
      @Override
      public int getOrder() { return 1; }
    }
    ```
  - @Order 와 Ordered 인터페이스는 동일한 동작을 한다.
  - BeanFactory 수준에 제어가 필요할 때 / 거의 없음

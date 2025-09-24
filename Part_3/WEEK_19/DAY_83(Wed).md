# DAY_83

## 목차
- [OAuth 워크플로우](#oatuh_워크플로우)
- [OpenID Connect](#openid_connect(oicd))
- [Spring Security](#spring_security)
- [서블릿 필터](#서블릿_필터)
- [주요 보안 필터](#주요_보안_필터)
- [필터 순서와 우선순위](#필터_순서와_우선순위)
- [인증 프로세스](#인증_프로세스)
- [인증 핵심 컴포넌트](#인증_핵심_컴포넌트)
- [인가 프로세스](#인가_프로세스)
- [인가 핵심 컴포넌트](#인가_핵심_컴포넌트)
---
## OAuth 워크플로우
- 사용자가 신뢰할 수 있는 제 3자 서비스를 통해 인증을 진행하고, 애플리케이션이 Access Token을 받아 Resource Server에 접근하는 과정
- 서비스 인증 요청 -> 인증 요청(리다이렉트) -> 로그인 및 동의 -> client로 Access Token 전달 -> 이후 resource Server와 client만 상호작용

#### Authorization Grant
- Client 애플리케이션이 Access Token을 얻기 위한 Resource Owner의 권한을 표현하는 Credential
- Client가 Access Token을 얻기 위한 수단
- 네 가지 유형
  - Authorization Code
  - Implicit
  - Client Credentials
  - Resource Owner Password Credentials
 
- Authorization Code Grant : 권한 부여 승인 코드 방식
  - 권한 부여 승인을 위해 자체 생성한 Authorization code를 전달하는 방식
  - Refresh Token을 사용할 수 있음
  - 권한 부여 승인 요청 시 응답 타입 (response_type)을 code로 지정하여 요청
 
- Implicit Grant : 암묵적 승인 방식
  - 별도의 Authorization Code 없이 바로 Access Token을 발급하는 방식
  - Refresh Token 사용 불가
  - 권한 부여 승인 요청 시 응답 타입 (response_type)을 token으로 지정하여 요청
 
- Resource Owner Password Credential Grant : 자원 소유자 자격 증명 승인 방식
  - 간단하게 로그인 시 필요한 정보로 Access Token을 발급받는 방식
  - 자신의 서비스에서 제공하는 애플리케이션의 경우에만 사용되는 인증 방식
  - Refresh Token 사용 가능
  - ex) 네이버 계정으로 네이버 웹툰 애플리케이션에 로그인, 카카오 계정으로 카카오 지도 애플리케이션 등
 
- Client Credentials Grant
  - Client 자신이 관리하는 Resource 혹은 Authorization Server에 해당 Client를 위한 제한된 Resource 접근 권한이 설정되어 있는 경우 사용

## OpenID Connect(OIDC)
- OAuth2는 인가에 초점을 둔다.
- OIDC는 OAuth2를 확장한 인증 프로토콜
- OAuth2의 Access Token 발급 과정을 더해 ID Token(JWT 기반)을 추가로 발급하여 사용자의 인증 정보를 제공
- 소셜 로그인의 기반
#### ID 토큰
- OIDC가 OAuth2를 확장하면서 추가된 핵심 요소
- JWT 형식으로 발급되며, 사용자 인증 결과와 기본 프로필 정보를 포함
  - 사용자 고유 ID, name, email, picture 등...
 
## Spring Security
#### 보안이 없는 애플리케이션의 문제점
- 로그인 기능이 없음
- API에 대한 권한 부여 기능이 없음
- 웹 보안 취약점에 대한 대비 부족

#### Spring Security.
- Spring MVC 기반 애플리케이션에서 인증과 인가 기능을 지원하는 보안 프레임워크
- 가능한 보안 기능
  - **다양한 인증 방식 지원**
  - **사용자 권한에 따른 접근 제어**
  - **리소스 접근 제어**
  - **민감한 정보 암호화**
  - SSL 연동 및 전송 구간 보안 강화
  - 알려진 주요 웹 보안 공격 차단
  - 추가적으로 SSO, 메서드 보안, Access Control List 같은 고급 기능도 제공

- 용어
  - Principal(주체) : 사용자
  - Authentication(인증) : 사용자가 본인이 맞음을 증명하는 절차
  - Credential(자격 증명 정보) : 사용자를 식별하는 정보
  - Authorization(인가/권한 부여) : 인증된 사용자에게 역할 또는 권한을 부여하는 특정 리소스 접근 허용
  - Access Control(접근 제어) : 사용자가 애플리케이션의 리소스에 접근하는 행위를 제어하는 과정

- Spring Boot 통합
  - 자동 설정 특징
  - 통합 장점
    - Spring Boot의 자동 설정 덕분에 초보자도 보안을 쉽게 적용 가능
    - SecurityFilterChain 등 주요 보안 컴포넌트를 Spring Boot가 자동으로 등록
    - 기존 Spring MVC 애플리케이션 구조와 자연스럽게 결합

#### 플랫폼 별 Spring Security
- 서블릿 기반
  - 서블릿 기반 애플리케이션은 동기,블로킹 I/O 모델로 동작
  - 요청이 들어오면 요청당 스레드가 할당되어 컨트롤러까지 진행
  - Spring Security는 이 경로 앞단에 FilterChainProxy를 두고 다수의 서블릿 필터를 순서대로 통과시키며 인증,인가,예외처리,CSRF 등을 수행
  - 필터 체인 -> 컨트롤러 순서이며, 모든 요청이 필터를 반드시 거침
  - 요청 경로 기반 : authorizeHttpRequests()에서 경로별 권한 지정
  - 메서드 기반 :@PreAuthorize("hasRole('ADMIN')") 등으로 서비스 계층에서 2차 검증
  - 기본은 세션 기반, 토큰 기반과 병행 시 세션 생성 정책 조정 필요
  - CSRF : 상태 유지형 POST 등에서 보호 필요. REST API만 제공하면 비활성할 수 있으나, 폼 제출이 있다면 반드시 활성 후 토큰 포함
  - 예외처리 : 인증 실패 -> `AuthenticationEntryPoint`, 인가 실패 -> `AccessDeniedHandler`
  - 요청당 스레드 고정으로 코드 단순성이 높음.
  - 높은 동시성에서는 스레드 풀이 병목 될 수 있어 캐싱,커넥션 풀, 쿼리 최적화가 중요
  - 경로 매칭 우선순위 점검 필요 (위에서 아래로)
 
- 리액티브 기반
  - 비동기,논블로킹 I/O와 리액티브 스트림 모델로 동작.
  - 요청은 이벤트 루프에 등록되어 스레드를 점유하지 않고 파이프라인을 타고 흐름.
  - 보안은 SecurityWebFilterChain(리액티브 웹 필터 체인)으로 설정
  - 세션 : 기본적으로 WebSession 기반, 토큰 기반과 혼용 시 ServerSecurityContextRepository 전략 검토
  - CSRF : 폼 제출 등 상태 유지형 요청에 필요. API만 제공하면 비활성 가능하나, 운영에서는 경로별 분리 권장
  - 성능 확장성 특징
    - 이벤트 루프 기반으로 적은 스레드로 높은 동시성 처리
    - 블로킹 I/O 호출 (전통 JDBC 등)이 섞이면 이점이 상쇄되므로 리액티브 스택 전반(DB, 외부 API)을 고려해야 함.
   
  - 블로킹 호출 존재 여부 점검 (리액터 스레드에서 블로킹 호출 금지)
  - pathMatchers 순서와 권한 매핑 확인

- 선택 가이드
  - MVC 경험 많고 JDBC 위주라면 서블릿이 유리
  - 다수 장기 연결, 스트리밍, WebSocket 중심이면 리액티브가 유리
  - 호출 대상 API/DB가 블로킹이면 서블릿이 단순

## 서블릿 필터
서블릿 기반 애플리케이션의 엔드포인트에 요청이 도달하기 전에 중간에서 요청을 가로채 특정 작업을 수행할 수 있는 Java컴포넌트
> dispatcherServlet 전에 동작

- Filter Chain
  - 여러 개의 Filter가 연결되어 체인을 이루는 구조
  - 요청이 들어오면 체인 형태로 나열된 Filter를 순서대로 통과함.
  - 모든 Filter 처리가 끝난 뒤 Servlet이 실행 됨
- 요청 URI path 기반 처리 : 서블릿 컨테이너는 요청 URI를 기반으로 어떤 Filter와 Servlet을 매핑할지 결정
- 순서 지정 가능 : Filter는 체인 내에서 실행 순서를 지정할 수 있음
- 요청과 응답 모두 Filter를 거침
- 필터는 서블릿 컨테이너 레벨에서 동작하기 때문에 Controller보다 먼저 실행 됨

## 주요 보안 필터
#### UsernamePasswordAuthenticationFilter
- 기본적으로 폼 로그인 요청을 처리하기 위해(id,password 기반 로그인 방식) 사용됨
- 클라이언트가 전달한 사용자 이름과 비밀번호를 HTTP 요청에서 추출
- 추출한 인증 정보를 기반으로 UsernamePasswordAuthenticationToken 객체 생성
- AuthenticationManager로 전달되어 실제 인증 절차 진행

#### BasicAuthentication
- HTTP Basic 인증 처리

#### DefaultLoginPageGeneratingFilter
- 사용자가 별도의 로그인 페이지를 구현하지 않은 경우, Spring Security가 기본 제공하는 로그인 페이지 생성

#### LogoutFilter
- 사용자의 로그아웃을 처리
- `/logout` 요청 감지
- 세션 무효화 수행
- SecurityContext에서 인증 정보 제거

## 필터 순서와 우선순위
- 서블릿 필터는 여러 개가 동시에 등록될 수 있으며, 이때 실행 순서에 따라 애플리케이션의 동작이 크게 달라짐
- 예를 들어, 인증 필터가 권한 부여 필터보다 뒤에서 실행된다면, 권한 검증을 수행할 수 없는 문제가 발생.
- 보안 필터의 경우, 인증 필터 -> 권한 부여 필터 -> 로깅 필터 순으로 실행되는 것이 일반적

## 인증 프로세스
<img width="1411" height="818" alt="image" src="https://github.com/user-attachments/assets/1e9a687b-a535-460a-81de-414e4548c0ac" />

#### 인증 처리 흐름
- 사용자가 Username(로그인 ID)과 Password를 포함한 request를 애플리케이션에 전송.
  - UsernamePasswordAuthenticationFilter가 전달받음
- UsernamePasswordAuthenticationFilter는 Username과 Password를 이용해 `UsernamePasswordAuthenticationToken`을 생성.
  - 이 시점에서 Authentication은 아직 인증되지 않은 상태
- 인증되지 않은 Authentication은 AuthenticationManager에게 전달
  - 실제 구현체는 ProviderManager
- ProviderManager는 인증처리를 직접 하지 않고, 등록된 여러 AuthenticationProvider 중 적절한 Provider에게 인증을 위임
- AuthenticationProvider는 UserDetailsService를 이용해 데이터베이스 등의 저장소에서 사용자의 정보를 조회
- 조회된 사용자 정보(UserDetails)에는 Username, 암호화된 Password, 권한 정보가 포함됨
- AuthenticationProvider는 PasswordEncoder를 이용해 입력된 비밀번호와 저장된 암호화된 비밀번호를 비교
  - 일치하면 인증에 성공한 Authentication 생성
  - 불일치하면 Exception 발생
- 인증에 성공한 Authentication은 ProviderManager를 거쳐 다시 UsernamePasswordAuthenticationFilter로 전달
- 최종적으로 인증된 Authentication은 SecurityContextHolder의 SecurityContext에 저장.
  - 이후 세션 정책에 따라 HttpSession에 저장되거나 무상태로 유지

#### 핵심 인증 컴포넌트
- AuthenticationManager
  - 인증 관리자의 역할
  - ProviderManager가 대표적인 구현체
  - 인증 위임 메커니즘
 
- ProviderManager
  - 다중 AuthenticationProvider 관리
    - 하나의 애플리케이션은 여러 인증 방식을 동시에 지원할 수 있음
  - 인증 처리 위임 메커니즘
  - 인증 예외 처리
 
- AuthenticationProvider
  - 인증 로직을 실제로 수행하는 컴포넌트
  - 사용자 정보 검증, 비밀번호 비교, 권한 확인 등의 로직 포함
  - 커스텀 AuthenticationProvider 구현
 
## 인증 핵심 컴포넌트
#### UsernamePasswordAuthenticationFilter
- 로그인 request를 제일 먼저 만나는 컴포넌트
- 일반적으로 로그인 폼에서 제출되는 Username과 Password를 통한 인증을 처리하는 Filter
- 인증되지 않은 UsernamePasswordAuthenticationToken 생성

#### AbstractAuthenticatinoProcessingFilter
- UsernamePasswordAuthenticationFilter가 상속하는 상위 클래스
- HTTP 기반의 인증 요청을 처리하지만 실질적인 인증 시도는 하위 클래스에 맡기고, 인증에 성공하면 인증된 사용자 정보를 SecurityContext에 저장하는 역할

#### UsernamePasswordAuthenticationToken
- Spring Security에서 Username/Password로 인증을 수행하기 위해 필요한 토큰
- 인증 성공 후 인증에 성공한 사용자의 인증 정보가 UsernamePasswordAuthenticationToken에 포함되어 Authentication 객체 형태로 SecurityContext에 저장

#### Authentication
- Spring Security에서의 인증 자체를 표현하는 인터페이스
- 인증을 위해 생성되는 인증 토큰 또는 인증 성공 후 생성되는 토큰은 하위 클래스로 생성되지만 생성된 토큰을 리턴 받거나 SecurityContext에 저장될 경우에 Authentication 형태로 리턴 받거나 저장된다.

#### AuthenticationManager
- 인증 처리를 총괄하는 매니저 역할을 하는 인터페이스

#### ProviderManager
- Spring Security에서 AuthenticationManager 인터페이스의 구현 클래스라고 하면 일반적으로 ProviderManager를 가리킴
- AuthenticationProvider를 관리하고, AuthenticationProvider에게 인증 처리를 위임하는 역할

#### AuthenticationProvider
- AuthenticationManager로부터 인증 처리를 위임받아 실질적인 인증 수행을 담당하는 컴포넌트

#### UserDetails
- 데이터베이스 등의 저장소에 저장된 사용자의 Username과 사용자의 자격을 증명해주는 크리덴셜인 Password 그리고 사용자의 권한 정보를 포함하는 컴포넌트
- AuthenticationProvider는 UserDetails를 이용해 자격 증명을 수행

#### UserDetailsService
- UserDatails를 로드하는 핵심 인터페이스

#### SecurityContext와 SecurityContextHolder
- SecurityContext는 인증된 Authentication 객체를 저장하는 컴포넌트이고, SecurityContextHolder는 SecurityContext를 관리하는 역할을 담당
- Spring Security 입장에서는 SecurityContextHolder에 의해 SecurityContext에 값이 채워져 있다면 인증된 사용자로 간주

## 인가 프로세스
<img width="1132" height="769" alt="image" src="https://github.com/user-attachments/assets/26ed0c3b-f859-4c9a-ab0c-b89c631a6b5d" />
#### 인가처리 흐름
- Spring Security Filter Chain 에서 URL을 통해 사용자의 액세스를 제한하는 권한 부여 Filter는 바로 AuthorizationFilter
- AuthorizationFilter는 먼저 SecurityContextHolder로부터 Authentication을 획득
- SecurityContextHolder로부터 획득한 Authentication과 HttpServletRequest를 AuthorizationManager에게 전달
- AuthorizationManager는 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스
  - RequestMatcherDelegatingAuthorizationManager는 AuthorizationManager를 구현하는 구현체 중 하나
  - RequestMatcherDelegatingAuthorizationManager가 직접 권한 부여 처리를 하는 것이 아니라 RequestMatcher를 통해 매치되는 AuthorizationManager 구현 클래스에게 위임만 한다
- RequestMatcherDelegatingAuthorizationManager 내부에서 매치되는 AuthorizationManager 구현 클래스가 있다면 해당 AuthorizationManager 구현 클래스가 사용자의 권한을 체크
- 적잘한 권한이라면 다음 요청 프로세스를 계속 이어감
- 만약 적절한 권한이 아니라면 AccessDeniedException이 throw되고 ExceptionTranslationFilter가 처리.

#### 컴포넌트 세부 흐름
- FilterSecurityInterceptor와 AuthorizationFilter
- 인가 결정 위임 메커니즘
- 인가 성공/실패 시 처리 방식

#### URL 기반 인가 설정
- requestMatchers().access() 메서드 활용
  - URL 요청 경로별로 권한을 제어할 수 있음.
  - RequestMatchers() 사용.
- 경로 매칭 전략(Ant/ MVC/ Regex)
  - Ant패턴 가장 흔함
    - `*` 경로 요소 1개. `**` 하위 모든 디렉토리 포함
  - MVC 패턴
  - Regex 패턴
- 정적 리로스와 API 경로 보호
  - 정적 리소스는 보통 permitAll()로 허용
 
#### 메서드 보안
- @EnableMethodSecurity 활성화
  - 위 설정을 통해 @PreAuthorize, @PostAuthorize 어노테이션 사용 가능
 
- @PreAuthorize, @PostAuthorize 활용
  - @PreAuthorize : 메서드 실행 전 인가 검사
  - @PostAuthorize : 메서드 실행 후 반환값 기반 인가 검사
 
- 메서드 보안은 SpEL을 사용하여 메서드 파라미터를 조건으로 활용할 수 있음.
- 컨트롤러나 서비스 메서드에 전달된 파라미터 이름은 SpEL에서 그대로 참조할 수 있음
- 여러 조건 조합 가능 `or` `and`
- 정리 포인트
  - `#파라미터이름` : 메서드 인자 그대로 사용 가능
  - `authentication` : SecurityContext의 Authentication 객체
  - `principal` : Authentication의 principal 보통 UserDetails
  - 복잡한 조건도 `and`,`or`로 조합 가능
    - ex) ```@PreAuthorize("hasRole('ADMIN') or #userId == principal.id")```
   
#### 인가 이벤트 처리
- AuthorizationEvent 발행
  - 인가 과정에서 이벤트가 발행되며, 이를 구독하여 로깅이나 모니터링을 수행할 수 있음
 
## 인가 핵심 컴포넌트
#### AuthorizationFilter
- URL을 통해 사용자의 액세스를 제한하는 권한 부여 Filter

#### AuthorizationManager
- 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스
- 인가 과정을 더 단순하고 직관적으로 구성하기 위해 등장한 컴포넌트.
- 함수형 인터페이스 기반 설계
  - 함수형 인터페이스로 정의되어 있어 람다 표현식으로 쉽게 작성 가능.
- and(), or()를 통해 권한 조건을 조합할 수 있음. 코드 가독성이 높아짐

#### 주요 AuthorizationManager 구현체
- RequestMatcherDelegatingAuthorizationManager
  - 직접 권한 부여 처리를 수행하지 않고 RequestMatcher를 통해 매치되는 AuthorizationManager 구현 클래스에게 권한 부여 처리를 위임
 
- AuthorityAuthorizationManager
  - 특정 권한이 있는지 검사
  - 간단한 권한 비교 시 자주 사용
 
- AuthenticatedAuthorizationManager
  - 인증 자체 여부만 검사

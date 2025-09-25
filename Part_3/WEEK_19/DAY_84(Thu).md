# DAY_84

## 목차
- [커스텀 필터](#커스텀_필터)
- [웹 보안 이슈](#웹_보안_이슈)
- [쿠키와 세션 기반 인증 복습](#쿠키와_세션_기반_인증_복습)
  - [세션 기반 인증 아키텍처](#세션_기반_인증_아키텍처)
- [SecurityContext](#securitycontext)
- [세션 관리 필터](#세션_관리_필터)
- [세션 이벤트](#세션_이벤트)
- [세션 생성 정책](#세션_생성_정책)
- [동시 세션 제어](#동시_세션_제어)
- [세션 고정 보호](#세션_고정_보호)
---

## 커스텀 필터
#### 커스텀 필터 필요성
- 특정 즈니스 로직이나 추가적인 보안 정책을 적용하려고 할 때, 내장 필터만으로는 한계가 있음
  - 모든 요청에 대해 추가적인 로깅을 수행하고 싶을 때
  - 특정 헤더 값이 반드시 포함되어야 하는 규칙을 적용하고 싶을 때
  - 사용자의 IP 주소를 검증하거나 화이트리스트 정책을 적용하고 싶을 때
- 활용 사례
  - 추가 인증 단계
  - 로깅 및 감사
  - 보안 정책 적용
  - 성능/모니터링
 
- 남용하면 성능 저하와 복잡도를 유발할 수 있음

#### 커스텀 필터 구현 방법
- GenericFilterBean 확장
  - 사용자 세션 만료 처리, IP 화이트리스트 등의 활용
  - 요청과 응답 모두 다ㄹ며, 전후 처리 제어가 필요한 경우 주로 사용 됨
 
- OnePerRequestFilter
  - 한 요청당 반드시 한 번만 실행되도록 보장
  - 주로 인증/인가 검증, 로깅, 헤더 검증 등의 작업에서 활용
    - 요청마다 반드시 한 번 실행되어야 하는 보안/로그/추적 로직에 적합
   
#### 필터 추가
- 특정 위치에 필터 추가
  - SecurityFilterChain 구성 시 `addFilterBefore`, `addFilterAfter`, `addFilterAt` 메서드 사용
    ```java
    @Configuration
    public class SecurityConfig {
    
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http
                .addFilterBefore(new CustomOncePerRequestFilter(), UsernamePasswordAuthenticationFilter.class)
                .authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
            return http.build();
        }
    }
    ```
- URL 패턴 기반 필터 적용
  - 단일 체인 + 내부 분기
    - 하나의 SecurityFilterChain만 존재
    - 모든 요청이 같은 필터 체인을 거침
    - 필터 내부에서 request.getRequestURI()를 검사하여 조건 분기
   
  - 다중 SecurityFilterChain 구성
    - URL 패턴별로 별도의 SecurityFilterChain을 정의

| 구분     | 방법 1 (단일 체인 + 내부 분기)         | 방법 2 (다중 체인)                  |
|---------|----------------------------------|---------------------------------|
| 실행 방식 | 모든 요청이 필터를 거치고 내부에서 URL 검사 | URL 패턴 매칭 후 해당 체인만 실행      |
| 장점     | 단순 구현, 빠른 적용	불필요한 요청        | 제외 → 성능 ↑                     |
| 단점     | 모든 요청 실행 → 오버헤드               | 필터 체인 관리 복잡도 ↑              |
| 추천 상황 | 간단 조건, 테스트/프로토타입             | 운영 환경, URL별 보안 정책 구분 필요 시 |


## 웹 보안 이슈
#### CSRF
- 사용자가 의도하지 않은 요청을 공격자가 위조하여 서버에 보내도록 하는 공격 기법
  - 결과적으로 사용자는 모르는 사이에 계정 정보 변경, 송금, 글 작성 등의 행위가 발생할 수 있음
- 서버는 요청 시 CSRF 토큰을 검증하여, 클라이언트가 의도적으로 요청을 보냈는지 확인
- CsrfFilter
  - 모든 요청에 대해 CSRF 토큰을 검증하는 필터
  - 사용자가 보낸 요청에 _csrf 파라미터나 헤더 값이 없는 경우 AccessDeniedException을 발생
 
- CsrfToken
  - 서버가 세션별로 생성하여 클라이언트에 전달하는 토큰
  - 이 토큰은 매 요청마다 클라이언트가 전달해야 하며, 서버는 저장된 토큰과 비교하여 일치 여부 검사
 
#### XSS
- 공격자가 악의적인 스크립트를 웹 페이지에 삽입하여 사용자의 브라우저에서 실행되도록 하는 공격 기법
- 주로 쿠키 탈취, 세션 하이재킹, 악성 사이트 리다이렉션, 피싱 등에 활용
- CSP와 nosniff방식 많이 사용
  - 브라우저가 응답 헤더를 근거로 위험한 리소스/스크립트를 차단하도록 하는 접근
 
- 컨트롤러에서 정확한 Content-Type 지정
- 전역으로 X-Content-Type-Options: nosniff 적용 : 브라우저의 임의 추측 금지

- CSP
  - 브라우저가 강제하는 보안 정책
  - 어떤 출처의 리소스만 로드/실행 가능한지를 선언적 헤더로 규정
  - nonce
    - 요청마다 서버가 난수를 생성 `<script nonce="<값>">`에만 실행 허용
    - 동적 스크립트에 유연
  - hash
    - 스크립트 내용의 SHA-256 등 고정 해시가 정책에 등록된 것만 실행 허용
    - CDN/정적 스크립트에 적합
   
#### CORS
- 교차 출처 리소스 공유를 의미
- 서로 다른 Origin을 가진 애플리케이션이 서로의 리소스에 접근할 수 있도록 해줌
- Origin : 프로토콜 + 호스트 + 포트의 조합
  - 브라우저에서만 발생
 
- 동일 출처 정책 (Same-Origin Policy)
  - 브라우저는 SOP를 적용하여 다른 출처의 리소스 접근을 차단
  - 외부 리소스를 무분별하게 가져오는 것을 막아, XSS,XSRF 공격을 방어할 수 있음
  - 실제 웹페이지는 외부 리소스를 자주 사용, 이를 위해 SOP 예외로 제공되는 메커니즘이 CORS
 
- 동작 원리
  - Simple Request
    - 서버에 바로 요청을 보내고, 서버는 Access-Control-Allow-Origin 헤더를 포함한 응답을 반환
    - 브라우저는 해당 헤더를 확인하여 요청 허용 여부를 결정
    - 요청 메서드는 GET, HEAD, POST 중 하나
    - Accept, Accept-Language, Content-Language, Content-Type 등 제한된 헤더만 사용 가능
    - Content-Type은 application/x-www-from-urlencoded, mulitpart/form-data, text/plain만 허용
   
  - Preflight Request
    - 본 요청 전에 OPTIONS 메서드로 예비 요청을 보냄
    - 서버가 허용 여부 Access-Control-Allow-* 헤더를 응답하면 브라우저가 본 요청을 진행
   
- 해결 방법
  - Access-Control-Allow-Origin
    - 브라우저가 리소스를 접근할 수 있는 출처를 지정
    - *(와일드카드)를 쓰면 모든 출처를 허용
  - Access-Control-Allow-Methods
    - 허용할 HTTP 메서드를 지정
  - Access-Control-Expose-Headers
    - 브라우저 JavaScript 코드에서 접근 가능한 응답 헤더를 지정
  - Access-Control-Allow-Headers
    - 브라우저가 보낼 수 있는 요청 헤더를 지정
  - Access-Control-Max-Age
    - Preflight 요청 결과를 캐싱할 시간을 지정
  - Access-Control-Allow-Credentials
    - 쿠키,인증 정보를 포함한 요청을 허용할 지 여부를 설정
   
## 쿠키와 세션 기반 인증 복습
- HTTP는 본질적으로 무상태(Stateless) 프로토콜
- 즉, 클라이언트가 보낸 요청과 그 이전 요청은 서버 입장에서 서로 연결되지 않음
- 사용자가 로그인 요청을 보내 인증되었다고 해도, 다음 요청에서 서버는 같은 사용자인지 알 수 없음
- 사용자의 인증된 상태를 유지하려면 상태 유지 메커니즘이 필요
  - 서버가 사용자 상태 정보를 저장하고, 이후 요청에서 이를 확인
 
#### 쿠키
- 브라우저는 이후 같은 도메인 요청 시 쿠키를 자동으로 함께 전송
- 구조
  - 이름 : 쿠키를 구분하는 키
  - 값 : 쿠키에 저장되는 실제 데이터
  - 경로 : 쿠키가 유효한 요청 경로
  - 도메인 : 쿠키가 전송될 도메인
  - 만료 시간 : 쿠키의 유효 기간
  - Secure : HTTPS에서만 전송
  - HttpOnly : 자바스크립트 접근 차단
  - SameSite
 
#### 세션
- 서버에서 관리하는 사용자 상태 정보 저장 공간
- 사용자가 로그인하면, 서버는 세션을 생성하고 고유한 세션 ID를 발급
- 세션 ID는 쿠키에 저장되어 클라이언트와 서버 간 인증상태를 이어줌

- 생명 주기
  - 클라이언트 로그인 요청
  - 서버는 인증에 성공하면 세션 객체 생성
  - 서버는 세션 ID를 쿠리로 발급해 클라이언트에 전송
  - 클라이언트는 이후 요청마다 세션 ID 쿠리를 함께 보냄
  - 서버는 세션 저장소에서 해당 세션 ID를 조회하여 사용자의 상태를 확인
  - 로그아웃 하거나 세션이 만료되면 세션은 제거 됨
 
#### 세션 기반 인증 아키텍처
- **UsernamePasswordAuthenticationFilter**: 로그인 요청을 가로채어 Authentication 객체를 만듭니다.
- **AuthenticationManager**: 인증 처리의 중심이며, 하나 이상의 AuthenticationProvider에 인증을 위임합니다.
- **AuthenticationProvider**: 실제 인증 로직(예: DB 조회, 비밀번호 비교 등)을 담당합니다.
- **SecurityContext**: 현재 인증된 사용자(Authentication)를 보관합니다.
- **HttpSession**: SecurityContext를 세션에 저장하여 요청 간 상태를 유지합니다.

#### 기본 세션 관리 동작
- 로그인 후 세션 생성 과정
  - 사용자가 로그인 폼에서 아이디와 패스워드 입력
  - 서버는 인증 절차를 수행하여 사요자 정보를 검증
  - 인증에 성공하면 서버는 HttpSession 객체 생성
  - 세션에는 사용자의 인증 상태 및 필요한 데이터 

- 로그아웃 시 세션 무효화
  - 로그아웃 요청이 들어오면 서버는 해당 세션을 제거
  - 브라우저에 저장된 쿠키는 더 이상 유효하지 않게 됨
  - 단순히 세션만 삭제하는 것이 아니라, 쿠키 제거도 함께 처리하는 것이 안전
 
- 세션 타임아웃 처리
  - 세션은 일정 시간 동안 사용되지 않으면 자동으로 만료
  - 기본값은 보통 30분
    ```yml
    server:
      servlet:
        session:
          timeout: 30m
    ```

## SecurityContext
#### SecurityContextHolder
- 현재 인증 정보를 저장하고 조회하는 중앙 저장소 역할
- 기본적으로 스레드 로컬 방식을 사용하여 각 요청마다 인증 정보를 독립적으로 보관

#### SecurityContext구조
- 실제 인증 정보인 Authentication 객체를 담고 있는 컨테이너

#### SecurityContextHolder 동작 방식
- 스레드 로컬 기반
  - 기본적으로 ThreadLocal을 사용하여 각 요청 스레드마다 별도의 SecurityContext를 저장
  - 이렇게 하면 멀티스레드 환경에서 인증 정보가 뒤섞이지 않고 요청 단위로 분리

- 전략 모드 변경
  - MODE_THREADLOCAL 기본값, 요청 스레드마다 독립된 SC 저장
  - MODE_INHERITABLETHREADLOCAL 자식 스레드가 부모 스레드의 SC를 상속
  - MODE_GLOBAL 모든 스레드가 하나의 SC를 공유
 
## 세션 관리 필터
- SecurityContextHolderFilter : 요청마다 SecurityContext 생성 및 복원
- SessionManagementFilter : 인증 시 세션 전략 적용 (세션 고정 보호, 동시 세션 제어 등)
- ConcurrentSesstionFilter : 이미 존재하는 세션의 유효성 검사 및 만료처리

#### SecurityContextHolderFilter
- 요청이 들어올 때마다 SecurityContext를 생성하거나 HttpSession에서 기존 Context를 복원
- 요청 처리 후 SecurityContextHolder.clearContext()를 통해 ThreadLocal에 저장된 Context 제거

##### HttpSessionSecurityContextRepository
- 기본 구현체로 HttpSession에 SecurityContext를 저장/조회
- 로그인 성공 시 인증 객체를 담은 SecurityContext가 세션에 저장 됨

#### SessionManagementFilter
- 로그인 시점에 동작하여 세션 인증 전략을 실행
- 주로 세션 고정 보호, 동시 세션 제어를 처리
- 세션 고정 보호
  - 공격자가 미리 발급받은 세션 ID를 피해자에게 심어주고, 로그인 후에도 동일한 세션을 공유하는 공격
  - Spring Security는 기본적으로 로그인 시 세션 ID를 변경하여 보호
- 동시 세션 제어
  - 한 사용자가 동시에 여러 개의 세션을 가질 수 없도록 제한
 
#### ConcurrentSessionFilter
- 요청이 들어올 때마다 세션의 유효성을 검사
- 세션이 만료되었거나 허용 개수를 초과한 경우, 사용자를 강제로 로그아웃 처리

## 세션 이벤트
- 세션의 생성, 소멸, 만료 시점에 발생하는 이벤트
- 리스너는 이러한 이벤트를 감지하고 특정 동작(로그 남기기, 리소스 정리 등)을 수행할 수 있도록 돕는 컴포넌트

#### HttpSessionListener
- 자바 EE 표준 인터페이스
- sessionCreated, sessionDestroyed 메서드를 통해 세션의 생성/소멸 감지

#### Spring ApplicationListener
- ApplicationEvent로 발행
- ApplicationListener를 구현하거나, @EventListener를 통해 간단히 감지할 수 있음
- HttpSessionCreatedEvent, HttpSessionDestroyedEvent 클래스를 통해 감지 가능

#### 세션 만료 감지
- 타임아웃 기반 만료
  - application.yml에서 세션 타임아웃 설정 가능
- ConcurrentSessionFilter 기반 만료
  - 동시 세션 제어와 함께 사용되며, 만료된 세션을 감지하고 처리
  - 세션이 만료되면 HttpSessionDestroyedEvent가 발생하며, 리스너에서 이를 받아 처리할 수 있음
 
## 세션 생성 정책
#### SessionCreationPolicy
- ALWAYS : 요청이 올 때마다 세션을 항상 생성
- IF_REQUIRED : 필요한 경우에만 생성
- NEVER : Spring Security는 세션을 생성하지 않지만, 기존 세션이 있으면 사용
- STATELESS : 세션을 전혀 생성하지 않으며, 기존 세션도 사용하지 않음 / **REST API,JWT 기반 인증**시 사용

#### 만료된 세션 접근 처리
- Spring Security는 기본적으로 InvalidSessionStrategy를 통해 만료된 세션 접근을 처리함
- 아무 설정이 없으면 `/login` 페이지로 리다이렉션
- 단순한 로그인 페이지로 이동하는 것만으로는 부족할 수 있음

- 세션 만료 시 리다이렉션 설정
  - http.sessionManagement() 설정에서 invalidSessionUrl 또는 InvalidSessionStrategy를 지정할 수 있음
 
- InvalidSessionStrategy 구현
  - onInvalidSessionDetected(HttpServletRequest request, HttpServletResponse response) 메서드 오버라이딩
 
## 동시 세션 제어
- 웹 애플리케이션은 기본적으로 하나의 계정으로 여러 기기나 브라우저에서 동시에 접속할 수 있다.
- 하지만 보안상 또는 비즈니스 정책상 이를 제헌할 필요가 있음.
- 필요성
  - 보안 강화
  - 정책 준수
  - 리소스 절약
 
- maximumSession() 동일 사용자가 동시에 가질 수 있는 세션의 최대 개수
- maxSessionsPreventsLogin() 최대 세션 수를 초과했을 때 동작 방식
  - true : 새로운 로그인 시도 차단
  - flase : 새로운 로그인 허용, 기존 세션 중 하나 만료
 
## 세션 고정 보호
- 세션 고정 공격은 공격자가 미리 발급받은 세션 ID를 피해자에게 강제로 사용하게 한 뒤, 해당 세션을 탈취하는 공격 기법
- 세션 고정 보호 전략
  - none : 기존 세션 그대로 유지
  - newSession : 새로운 세션을 생성하고 기존 속성을 복사하지 않음
  - migrateSession : 기본값, 새로운 세션을 생성하고 속성을 복사
  - changeSessionId : 세션ID만 새로 발급, 속성 그대로 유지

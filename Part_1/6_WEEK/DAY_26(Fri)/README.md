# DAY_26

- [HTTP](#http-hypertext-transfer-protocol)
- [서블릿(Servlet)](#서블릿)
- [MVC](#mvc)
- [Spring MVC](#spring-mvc)

## HTTP HyperText Transfer Protocol
- application Layer에 속하는 프로토콜
- 웹 브라우저와 웹 서버 간의 통신을 위한 표준
- 클라이언트가 요청을 보내면 서버는 응답을 돌려주는 방식
- **무상태성**을 특징으로 하며 각각의 요청은 서로 독립적이다.**(비연결성)**
- 많은 클라이언트와 웹 서버 간에 데이터를 교환하고 통신하기 위한 표준 규칙 **TCP/IP**
- HTTP는 TCP/IP의 하위 항목 중하나로 html 같은 문서를 주고 받을 때 사용하는 규칙
- 하나의 html 파일에서도 웹 서버에 여러 개의 요청을 보낸다. ( 문서를 담는 서버, 영상을 담은 서버, 광고를 담는 서버 ... )
- http에서 각 요청은 독립적이여서 같은 컨텍스트 임에도 새로운 연결을 시도하기 때문에 성능 문제로 이어질 수 있다.

#### http 요청(request) 구조
- (http 요청 메시지)
- Start Line (요청줄)
  - *GET /products HTTP/1.1*<br>
  - method : 클라이언트가 수행하려는 작업 종류 ( get, post ... )
  - request Target : 요청 대상의 경로 ( ex: /products )
  - http version : 사용 중인 http 프로토콜 버전 ( ex: HTTP/1.1 )
- headers (요청 헤더)
  - key: value 쌍으로 이루어짐
  - 커스텀 키 가능
  - 요청에 대한 추가 정보를 제공
    - General : 메시지 전체에 영향을 줌 / Connection: keep-alive
    - Request : 브라우저, 클라이언트 정보 / User-Agent, Accept
    - Representation : Body의 포맷 지정 / Content-Type 요청을 보낼 때 데이터를 어떤 형식으로(JOSN,XML...)
- Body (본문)
  - 헤더를 제외한 모든 정보를 보낼 수 있다
  - get : 보통 body 없음
  - post, put : JSON, XML 파일 등 포함 가능
 
#### HTTP 응답(response) 구조 
- (http 응답 메시지)
- Status Line (상태줄)
  - *HTTP/1.1 200 OK*<br>
  - version : HTTP/1.1
  - Status Code : 200
  - Reason Phrase : OK (description)
- headers (응답 헤더)
  - General
  - Response
  - Representation
 
- Body (본문)
  - HTML, JSON, XML, 파일 등 다양하게 존재 가능
  - 204 No Content, 304 Not Modified 응답에는 Body가 없을 수 있다.

#### 주요 메서드
- GET
  - 리소스 조회
  - Body 없음, 캐싱 가능 (브라우저에서 캐싱 가능)
    - 실시간성을 일부 포기하면서 성능 향상을 얻음
  - 게시글 목록 조회, 유저 정보 요청
- POST
  - 리소스 생성
  - Body 필수, 중복 요청 시 동일 리소스 생성 가능
  - 게시글 작성, 회원 가입
  - GET, PUT, PATCH, DELETE로 표현하지 못하는 것을 POST로 많이 처리 함
- PUT
  - 리소스 **전체** 수정
  - 대상이 없으면 생성됨 (idempotent)
  - 게시글 전체 내용 수정
- PATCH
  - 리소스 **일부** 수정
  - 일부 필드만 전송 가능
  - 게시글 제목만 수정, 비밀번호 변경
- DELETE
  - 리소스 삭제
  - Body 거의 없음
  - 게시글 삭제, 회원 탈퇴

#### GET VS POST
- 요청 목적
  - GET : 데이터 조회
  - POST : 데이터 생성, 제출
- 요청 데이터 위치
  - GET : URL 쿼리 스트링
  - POST : HTTP Body(JSON/Form 등)
- 요청 본문
  - GET : 없음
  - POST : 있음
- URL 길이 제한
  - GET : 있음 (2048자)
  - POST : 없음 (이론상 무제한)
- 캐싱
  - GET : 가능 (브라우저/프록시 서버)
  - POST : 기본적으로 불가능
- 멱등성
  - GET : O (같은 요청 반복해도 동일 결과)
  - POST : X (중복 요청 시 리소스 중복 생성 가능)
- 브라우저 뒤로가기/새로고침
  - GET : 안전
  - POST : 새로고침 시 재요청 주의 (재등록 가능성)
- 보안
  - GET : 상대적으로 낮음 (주소에 데이터 노출)
  - POST : 상대적으로 높음

#### 멱등성
- 같은 연산을 여러 번 수행해도 동일 결과
- GET : O / 같은 리소스 요청이므로 여러 번 호출해도 결과가 같음
- PUT : O / 같은 자원에 같은 값을 덮어 씀
- DELETE : O / 이미 삭제된 리소스를 또 삭제해도 상태는 같음 -> 삭제되었다 복구되었다 하지 않음
- POST : X : 매 호출마다 새로운 리소스를 생성할 수 있음
- PATCH : X(일반적으로) : 일부 필드만 수정 -> 여러 번 수정 시 상태 달라질 수 있음 ( ex: count++ )

GET /search?**q=spring** HTTP/1.1 **쿼리 파라미터**
  + 필터링, 정렬
  + N개의 결과가 나올 수 있는 경우
GET /login**/1** HTTP/1.1 **쿼리 스트링**
  + 특정자원 접근
  + 유니크값 접근 -> 1개 보장

로그인 : 인증 식별 정보를 생성 하기 때문에 POST 사용 / +주소에 민감 정보 노출

#### HTTP 상태 코드
- 서버의 처리 결과를 숫자와 간단한 메시지로 나타낸 것
- 1xx 정보전달
- 2xx 요청 처리 성공
- 3xx 리다이렉션 (다른 위치로 이동 유도)
- 4xx 클라이언트 오류(요청 문제)
- 5xx 서버 오류(서버 내부 문제)

- 2xx
  - 200 OK 요청 성공
  - 201 Created 리소스 생성됨
  - 204 No Content 본문없음
 
- 3xx
  - 301 Moved Permanently 영구 이동
  - 302 Found 일시적 이동
  - // 304 Not Modified 변경 없음
 
- 4xx
  - 400 Bad Request 잘못된 요청
  - 401 Unauthorized 인증 실패
  - 403 Forbidden 권한 없음 (인증은 되었으나 접근 권한이 없음)
  - 404 Not Found 존재하지 않음
  - 409 Conflict 충돌
  - 415 Unsupported Media Type 지원하지 않는 형식 (Content-Type이 잘못된 경우)
  - // 429 Too Many Requests 요청 과다
 
- 5xx
  - 500 Internal Server Error 서버 내부 오류
  - 502 Bad Gateway 게이트웨이 오류 (프록시 서버에서 백엔드 서버 응답 문제 발생)
  - 503 Service Unavailable 서비스 불가
- 기본적으로 x00 가능하나 더 명확하게 표현하는 것이 좋음


#### Content-Type / Accept
**클라이언트가 서버에게**
  - Content-Type : 보내는 데이터 타입
  - Accept : 받을 데이터 타입
- @PostMapping( consumes, produces)
  - consumes = 받는 포맷
  - produces = 보내는 포맷
- 주요 content-type 정리
  - application/json
  - multipart/form-data


## 서블릿
- java 언어로 작성된 서버 측 웹 컴포넌트 -> 로우레벨에서 서블릿 API가 사용된다
- 프로세스가 아닌 스레드 기반 처리로 고성능 제공
- 객체지향 방식으로 코드 구조화 가능
- 세션, 쿠키, 리다이렉션 등 HTTP 기능을 손쉽게 사용 가능
- 다양한 WAS에서 표준적으로 작동 (추상화 되어 있음)

- doGet(), doPost() HTTP GET, POST 요청을 처리하는 메서드

#### 서블릿 생명주기
1. 생성자 호출 / 인스턴스가 최초로 생성됨
2. init() / 최초 요청 시 1회 호출, 초기화 로직 수행
3. service() / 클라이언트의 요청이 들어올 때 마다 실행, GET,POST 구분 없이 처리
4. doGet(), doPost() / service() 내부에서 HTTP 메서드에 따라 분기 실행
5. destroy() / WAS 종료 시 또는 서블릿이 제거 될 때 1회 호출, 자원 정리 용도
- 싱글톤 구조로 동작하기 떄문에 스레드 간 공유 자원에 주의
- destroy()로 자원 정리를 해야 메모리 누수를 방지

#### 서블리 컨테이너 역할
- 서블릿 클래스 관리
- 요청-응답 처리
- 멀티 스레드 처리
- 매핑 처리
- 보안 및 세션 관리
- 예외 및 에러 처리

주요 서블릿 컨테이너
- tomcat
- jetty
- netty
- resin

## MVC
- 웹 계층을 담당하는 모듈 spring-webmvc / spring-boot-starter-web에 내장
- 서블릿 API를 기반으로 클라이언트의 요청을 처리
- 웹 프레임워크의 한 종류

#### MVC 패턴
- Model : 데이터 및 비즈니스 로직 처리 ( service + repository + entity )
- View : 사용자에게 보여지는 화면 처리 ( 화면 생성 및 반환 )
- Controller : 요청 처리 및 흐름 제어

#### Model
- 클라이언트의 요청을 처리한 결과 데이터를 담고 있는 영역
- service layer에서 만들어진 객체가 곧 Model 데이터

#### View
- Model 데이터를 기반으로 사용자에게 시각적으로 보여지는 결과를 생성
- HTML, JSON, 문서(PDF, Excel 등) 생성
- SPA(Single Page Application) -> SEO 불가

#### Controller
- 요청을 직접적으로 수신하는 엔드포인트
- 응답도 처리
- **DTO** 또는 Entity 객체만 반환해야 함 / 거의 다 DTO

#### MVC 패턴의 장점
- 관심사의 분리
  - 책임과 목적에 따라 코드를 분리
  - Model, View Controller로 분리
 - 관심사 분리 효과
   - 코드 변경 범위 최소화: 화면ui만 바뀌는 경우 view만 수정하면 됨
   - 코드 재사용성 증가 : Model은 다양한 Controller와 view에서 재사용이 가능
   - 개발 역할 분담 : 명확한 협업 구조 가능
 - 유지보수성과 확장성 향상
 - **테스트 용이성**
   - 각 계층이 분리되어 있으면 단위 테스트가 쉬워짐
   - 단위 테스트 작성 시 Mocking이 용이


## Spring MVC
![image](https://github.com/user-attachments/assets/71e9a0c2-8b54-4205-90fe-aa859119cb6f)
- DispatcherServlet (프론트 컨트롤러 패턴의 구현체)
  - 주요 역할
    - 요청 수신 : 클라이언트가 전송한 모든 HTTP 요청을 가장 먼저 수신
    - 핸들러 검색 요청 : 요청을 처리할 Controller를 찾기 위해 HandlerMapping에 위임
    - 핸들러 실행 위임 : Controller의 메서드 실행을 위해 HandlerAdapter에 위임
    - 응답 렌더링 제어 : 처리 결과 (Model과 View)를 받아 ViewResolver와 View를 통해 응답을 생성
- HandlerMapping (요청과 핻늘러 매핑)
  - 클라이언트 요청 URI와 이를 처리할 Controller 메서드를 연결(매핑)해주는 역할
  - HTTP 메서드(GET, POST 등), 패턴 매칭, 파라미터 조건까지 고려
- HandlerAdapter (핸들러 호출 추상화)
  - 다양한 핸들러 타입을 일관된 방식으로 실행할 수 있도록 추상화
  - 다양한 핸들러 유형 지원 (Controller, @RestController 등)
- ViewResolver (뷰 이름 -> 실제 View 반환)
  - Controller가 반환한 View이름을 토대로 실제 View를 찾아주는 역할
- HttpMessageConverter (객체 -> HTTP 메시지 변환)
  - API 응답은 View가 아니라 직접 JSON, XML 등으로 데이터를 반환
  - Controller에서 반환한 객체를 HTTP Response Body에 맞게 변환
  - 역변환도 가능

#### Spring boot MVC
- spring-boot-starter-web : 웹 애플리케이션 개발에 필요한 핵심 의존성들을 묶어 놓은 스타터
- 기능
  - Spring MVC 웹 프레임워크
  - 내장 Tomcat 서버
  - Jackson을 이용한 JSON 처리
  - 정적 리소스 제공
  - 예외 처리 자동화
- 자동 설정 컴포넌트
  - DispatcherServlet 등록 및 URL 매핑
  - @RestController, @RequestMapping 자동 인식
  - MessageConverter 설정
  - ~~예외 처리용 BasicErrorController 등록~~
  - 정적 리소스 핸들링
- @getMapping("/home"), @postMapping("/home") 등으로 메서드와 엔드포인트로 자동 매핑

## Postman

#### 테스트 환경 구성
- 컬렉션 : 관련된 API 요청들을 하나의 그룹으로 묶어 관리할 수 있는 단위
  - postman에서 요청들을 구조적으로 정리하기 위한 컨테이너
- 컬렉션 활용 이점
  - 테스트 시나리오 분류
  - 일괄 실행
  - 협업 용이
  - 문서화 지원
- 환경 변수 사용하기
  - 요청을 더욱 유연하게 구성하기 위해 변수를 사용할 수 있다
  - 서버 주소, 토큰 등 환경별로 달라지는 값을 효율적으로 관리할 수 있다.
  - {{base_url}} 로 사용 가능
  - 변수의 종류
    - Global : 전체 요청
    - Environment : 특정 환경
    - Collection : 컬렉션 내부
    - Local : 한 요청 내

## Controller
#### API 계층
- 클라이언트의 요청을 직접적으로 전달받는 계층

#### 패키지 구조
- 기능 기반 패키지 구조 (도메인 기반 패키지 구조)
  - 하나의 기능을 완성하기 위한 계층별 클래스들이 모여 있다. (entity가 해당하는 api계층, 서비스계층, 데이터 액세스 곛층이 하나의 클래스에 모여있다.)
- 계층 기반 패키지 구조
  - 하나의 계층으로 보고 클래스들을 계층별로 묶어서 관리하는 구조








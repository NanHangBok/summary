## DAY_27

SpringBoot 엔트리 포인트
+ @SpringBootApplication
+ SpringApplication.run();


## 핸들러 메서드
- 클라이언트의 요청을 처리할 메서드
  
- @RequestMapping("/v1/members")
  - 클래스 레벨에서 공통 URI 경로를 지정하는 애너테이션
  - 해당 컨트롤러 내에 있는 모든 핸들러 메서드들은 /v1/members를 기준 URI로 가지게 된다.
  
- @PostMapping()
  - 클라이언트의 요청 데이터(Request Body)를 서버에 생성할 때 사용하는 애너테이션
  - 클라이언트 쪽에서 요청 전송 시, HTTP Method 타입을 동일하게 맞춰주어야 함.
  
- @RequestParam
  - 핸들러 메서드의 파라미터 종류 중 하나
  - 쿼리 파라미터, 폼 데이터, x-www-form-urlencoded 형식으로 전송하면 이를 서버 쪽에서 전달받을 때 사용하는 애너테이션

- Spring MVC에서는 컨트롤러 메서드가 반환하는 String 값을 View 이름으로 간주
  
- @GetMapping
  - 클라이언트가 서버에 리소스를 조회할 때 사용하는 어노테이션
  
- @PathVariable
  - 핸들러 메서드의 파라미터 종류 중 하나
  - 괄호 안에 입력한 문자열 값은 @GetMapping("/{member-id}") 처럼 중괄호 안의 문자열과 동일해야 함

- 요약
  - 요청 처리에는 핸들러 메서드가 필요하며, SSR(server side rendering) 방식에서는 view 이름을 반환
  - @RequestParam으로 form/x-www-form-urlencoded 데이터를 받음
  - @PathVariable을 통해 URL 경로 내 값을 받음
  - Model 객체를 활용하여 view로 값 전달

## 핸들러 메서드 파라미터
- @RequestParam : 쿼리 스트링 또는 form 필드 값 처리
- @PathVariable : URL 경로의 변수 값 처리
- @RequestHeader : 요청 헤더 정보 처리
- @CookieValue : 쿠키 정보 처리
- @RequestBody : 요청 본문 처리
- @ModelAttribute : 폼 데이터를 객체로 바인딩

- @RequestHeader
  - HTTP 요청에 요청 헤더 정보도 포함, 서버에서는 이를 활용해 사용자 환경에 맞춘 응답을 줄 수 있음
  - 커스텀 헤더도 작성 후 받을 수 있음
  - 여러 header를 동시에 받을 수도 있음
  - 헤더 전체도 받을 수 있음 @Request Map<String,String> headers로 받을 수 있음

- @CookieValue
  - 서버에서 쿠키를 생성해서 클라이언트에 전달할 수 있다.
  - cookie.setHttpOnly(true) -> 쿠키를 js로 수정이 가능하기 때문에 Http에서만 사용하겠다 ( 수정하지 못하도록 )
  - cookie.setMaxAge(3600) -> 시간이 지나면 쿠키가 자동 소멸한다.

- @RequestBody
  - 클라이언트에서 전달받은 JSON 정보를 토대로 DTO를 생성해서 자동으로 매핑한다.
  - Dto에 변수 명을 토대로 JSON 파일에 key값과 같은 이름을 자동 매핑한다.
  - Jackson 역직렬화 필수 2가지 / 1. 기본생성자 2. getter or Setter
 
- @*Mapping
  - 내부적으로 RequestMapping 사용
  - @*Mapping(method = RequestMethod.POST, value="/json") 가능
 
- @ModelAttribute
  - 클라이언트가 전송한 폼 데이터를 자바 객체로 자동으로 매핑해주는 어노테이션
  - 주로 HTML <form> 태그로 전송된 데이터를 처리할 때 사용
  - Spring MVC에서는 @ModelAttribute 생략해도 POST 방식의 form 데이터는 자동으로 객체에 바인딩 됨

- consumes / produces 설정
  - 서버가 어던 형식의 데이터를 받아들이고, 어떤 형식으로 응답할지 지정할 수 있음
  - consumes : 요청 데이터 타입
  - produces : 응답 데이터 타입

- JSON 처리 시 RequestBody, 폼 처리 시 @ModelAttribute, 인증 시 @RequestHeader, @CookieValue 등을 조합하여 사용

## 파일 업로드 처리
- multipart/form-data 라는 HTTP 요청 형식으로 전송
- 한번의 요청으로 텍스트 데이터와 바이너리 파일을 함께 서버에 전달할 수 있도록 설계
1. 사용자가 <input type="file">를 통해 로컬 파일을 선택
2. 사용자가 <form> wpcnf -> 브라우저는 요청의 Content-Type을 multipart/form-data로 설정
3. 각 form 필드와 파일 데이터는 서로 다른 파트로 분리되어 전송 됨.
- List<multipartForm> 으로 여러개의 파일을 받을 수 도 있음

## 응답처리
- ViewResolver, View
  - String 반환 값을 토대로 실제 템플릿 파일로 변환
  - model 안에 Attribute로 Html에 값을 삽입하여 View 를 완성하는게 View
  - application.properties, .yaml를 통해 위치나 확장자 선택 가능

- HttpMessageConverter
  - 객체를 JSON, XML 등으로 변환
  - @ResponseBody가 붙으면 ViewResolver를 사용하지 않음
  - 자동으로 HttpMessageConverter가 작동하여 객체를 JSON으로 변환
  - 객체/문자열 직접 반환

- @ResponseBody
  - 반환된 데이터를 HTTP 응답 본문에 직접 작성ㄹ하게 만든다
  - View를 거치지 않기 때문에 REST API 개발 시 주로 사용 됨
  - 내부적으로 RequestMappingHandlerAdpater -> HttpMessageConverter 순서로 호출
  - 클래스 단위에 어노테이션 사용 시 해당 클래스 내 모든 핸들러 메서드에 적용


- Model 기반 SSR 응답
  - 폼 데이터를 Model 객체에 담아 전달
  - 렌더링 시, ${name} 등의 표현식으로 Model의 데이터를 출력

- Map 객체 사용
  - Json으로 자동 변환
  - 기본적으로 Jackson 라이브러리가 포함되어 있음 -> Java 객체 -> Json 변환이 자동으로 이루어짐
  - JSON이 k-v 형태이기 때문에

- DTO 필드
  - @JsonInclude(JsonInclude.Include.NON_NULL) = null 일때 안보여 주게 가능 / @JsonInclude private Long age;

#### ResponseEntity
- 상태코드 포함
- 본문, 상태코드, 헤더를 모두 조작할 수 있다
```java
@PostMapping("/response")
    public ResponseEntity<Map<String, String>> postMemeberResponse(@RequestParam("email") String email,
                                                                   @RequestParam("name") String name,
                                                                   @RequestParam("phone") String phone) {
        Map<String, String> response = new HashMap<>();
        response.put("email", email);
        response.put("name", name);
        response.put("phone", phone);

        return new ResponseEntity<>(response, HttpStatus.CREATED);
    }
// public ResponseEntity postMemberResponse도 가능
```
- Header
  ```java
  HttpHeaders headers = new HttpHeader();
  headers.set("Custom-Header","Custom-value");
  return new ResponseEntity<>(responseBody, headers, HttpStatus.OK);
  ```
- HttpHeaders = MultiValueMap의 구현체 1:N Map일 때 사용 Map<String,List<>>
- 응답 본문 설정 (단축 메서드)
  - return ResponseEntity.ok(responseBody);
  - return ResponseEntity.status(HttpStatus.CREATED).body(responseBody);
  - return ResponseEntity.badRequest().body(errorBody);
  - return ResponseEntity.created = 보통 주소값이나 키 값을 반환(URI location) -> Map불가
 
- ResponseEntity<Resource> 또는 ResponseEntity<byte[]>
  - body에 파일을 추가해서 보내면 다운로드 가능
  - Resource resource = new InputStreamResource(Files.newInputStream(filePath)); -> ResponseEntity.body(resource)
  - 파일명이 한글일 경우 브라우저가 깨지지 않도록 인코딩 처리 필요

## 예외처리
- ResponseBody의 내용 만으로는 요청 데이터 중에서 어떤 항목이 유효성 검증에 실패했는지 알 수 가 없다.
- 클라이언트 쪽에서 에러메시지를 조금 더 구체적으로 친절하게 알 수 있도록 바꾸는 작업이 필요함
#### @ExceptionHandler
- 특정 예외가 발생했을 때, 해당 예외를 처리할 메서드를 지정할 수 있는 어노테이션
- @Contoller, @RestController 내부에서 사용되며, 특정 컨트롤러에서만 예외를 처리하고 싶을 때 유용
  ```
  @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<String> handleMemberNotFound(MemberNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(e.getMessage());
    }
  ```
- 문자열이 아닌 JSON 형식의 오류 메시지를 응답하는 경우가 많다.
- ErrorResponse {}

#### ControllerAdvice
- 여러 컨트롤러에서 공통적으로 발생할 수 있는 예외를 하나의 클래스에서 처리할 수 있도록 해주는 기능
- 전역에서 발생하는 예외를 처리하는 클래스
- 서비스에서 발생하는 예외는 컨트롤러로 가고 디스패처서블릿이 받아서 @ControllerAdvice를 찾아서 예외처리를 함.



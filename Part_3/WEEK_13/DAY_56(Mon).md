# DAY 56

## 목차
- [API 계층 테스트](#api-계층-테스트)

---

## API 계층 테스트
DispatcherServlet이 필요한데 무거움 -> MockMvc 지원 (@AutoConfigureMockMvc)
```
void 사용자 단건조회() throws Exception {
  //given
  User user = new User(1L, "Kim", "k@a.com");
  given(userService.findById(1L)).willReturn(user);

  //when

  // then
  mvc.perform(get("/api/users/{id}", 1L))
      .andExpect(status().isOk())
      .andExpect(jsonPath("$.id").value(1L))
      .andExpect(jsonPath("$.name").value("Kim"));
  then(userService).should().findById(1L);
}
```
- 요청 처리의 정확성 검증
  - Controller가 경로/메서드/파라미터를 올바르게 해석하고 서비스로 정확히 위임하는지 확인
 
- 입력값 검증
  - Bean Validation으로 필수/형식/길이 규칙을 선언하고, 검증 실패 시 400 + 오류메시지를 보장하는지
 
- 응답 상태와 형식 검증
  - 성공/실패 상태 코드와 JSON 스키마가 명세와 동일한지 확인
 
- 예외 처리 검증
  - 글로벌 예외 처리기가 도메인/검증 예외를 일관된 에러 형식으로 변환하는지 확인
 
- 테스트 해야 할 것
  - 요청 본문 검증이 필요한 엔드포인트
  - 커스텀 예외 처리가 있는 엔드포인트
 
- 테스트 하지 말아야 할 것
  - 단순 서비스 호출 위임 메서드
  - 단순 DTO 변환 메서드 / 헬퍼클래스,유틸클래스는 단위테스트 (Mapper 포함)
 
예제 코드
```
@Test
void postMember_이메일누락시_400반환() throws Exception {
    MemberDto.Post post = new MemberDto.Post("", "홍길동", "010-1234-5678");
    String content = gson.toJson(post);

    mockMvc.perform(post("/v11/members")
            .contentType(MediaType.APPLICATION_JSON)
            .content(content))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.code").value("VALIDATION_ERROR"))
        .andExpect(jsonPath("$.message").value(containsString("이메일은 필수입니다.")));
}
```
```
@Test
void getMember_없는ID조회시_404반환() throws Exception {
    mockMvc.perform(get("/v11/members/{id}", 9999)
            .accept(MediaType.APPLICATION_JSON))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.code").value("MEMBER_NOT_FOUND"));
}
```

- API 계층 테스트 시 확인해야 할 체크리스트
  - 성공/실패 케이스를 모두 작성했는가
  - 상태코드가 API 명세와 일치하는가
  - 응답 본문이 계약과 동일한가
  - 전역 예외 처리 응답 형식이 일관되는가
  - 테스트 데이터는 격리되는가?(@Transactional 롤백 여부)
 
- API 슬라이스 = @MockBean
- 서비스 테스트 = @Mock + @InjectMocks

## E2E 테스트
- 순서 중요
- 단일 시나리오를 끝까지 검증하는 것
- 장점
  - 실제 배포 환경과 유사한 조건에서 **기능의 완전성을 검증**
  - API/웹 서버/DB/캐시/서드파티 연동까지 사용자 관점 흐름을 확인
 
- 한계
  - 느리고, 때때로 불안정
  - 실패 시 원인 파악 비용이 큼
 
- 결론
  - 많이가 아니라 의미있게 : **핵심 사용자 여정 위주**로 적게, 정확히 운영
  - 나머지 대부분은 단위/슬라이스/통합 테스트에서 커버
  - 실패 시 단위 테스트부터 다시 검증하는 것이 일반적
 
- 목적 : Controller -> Service -> Repository까지 HTTP 레벨에서 실제 실행 경로를 검증
- 범위 : 톰캣 내장 서버를 띄우고 실제 포트로 요청을 보내며, 결과로 내려오는 상태 코드, 헤더, 본문을 계약으로 고정
- 가치 : 단위/슬라이스 테스트가 놓칠 수 있는 스프링 MVC 바인딩, Jackson 직렬화/역직렬화, 트랜잭션 경계, JPA 플러시 타이밍 문제를 조기에 발견

- 환경과 도구
  - DB : H2
  - HTTP 클라이언트 : TestRestTemplate, RestAssured
  - 서버 모드 : `@SpringBootTest(webEnvironment = RANDOM_PORT)`
  - 로깅 : 테스트 프로파일에서 도메인 로그만 debug, 나머지는 warn
 
## TDD
- 테스트 주도 개발
- Red : 실패하는 테스트를 먼저 작성하여 기능 요구사항을 명확히 한다.
- Green : 테스트를 통과할 수 있을 정도로 최소한의 기능을 구현
- Refactor : 테스트가 통과한 코드를 리팩토링하여 가독성과 유지보수성을 높인다.
- -> 추가 기능이 필요할 시 기능 추가 **분할정복**

- 장점
  - 요구사항 분석이 선행되어 명확한 개발 가능
  - 테스트 커버리지가 높아져 안정적인 코드 제공
  - 리팩토링이 안전하게 가능
  - 디버깅 시간 단축
  - **분할 정복에 적합한 구조 유도**
  - 결함 발견 시점이 빨라짐
 
- 단점
  - 초기 개발 속도가 느릴 수 있음
  - 모든 상황에 TDD가 적합하지 않음
  - 경험 부족 시 테스트 품질 저하
  - 테스트 코드 유지에 리소스 소요
 
- 장기적으로 품질과 유지보수 비용을 낮추는 전략
- 핵심 로직 처리 시 적용 권장

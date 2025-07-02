# DAY_29

## 프로젝트 진행사항
- API 테스트 후 오류 수정
- 코드 리팩토링
- 검증 로직 추가
- GlobalExceptionHandler 상황에 맞게 HttpStatus 수정
- 심화 요구사항 ( 정적 리소스 서빙 )
  - ApiController 추가 (@Controller 사용/ @RestController <- X)
  - View (웹 페이지) 추가
  - BinaryContent 필드 수정 -> 웹 페이지에 매핑 시키기 위해서

- 이슈 토픽
  - 직렬화 과정에서 MultipartFile을 직렬화 불가능
    - byte[] 로 필드 수정







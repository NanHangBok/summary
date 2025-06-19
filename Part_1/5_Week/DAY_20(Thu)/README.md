# DAY_20

## DTO
**도메인 객체를 전달하면 민감 정보 노출 등의 문제가 발생할 수 있다**<br>
Data Transfer Object -> 컨트롤러가 클라이언트에게 필요한 정보만 전달
- 컨트롤러에서만 사용하는 것이 보편적
- POJO 객체이다. (순수한 자바 객체)
- 최대한 불변을 유지해야 한다 (setter 안씀)
- record 클래스 사용하기도 함
  - RecordDto ( string id, string password) {}
  - 내부적으로 필드는 private final임
  - Getter가 내장되어 있음
  - 기본 생성자 없음, 모든 매개변수를 넣어야 함.
  - 필드명으로 조회 ( ex : recordDto.id() )

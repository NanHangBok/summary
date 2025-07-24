# DAY_44

## 프로젝트 진행사항
- ReadStatus를 읽지 못하고 새로 생성하는 오류 해결
  - ReadStatusService의 FindAllByUserId에서
  - 결과를 담을 리스트와 stream으로 toList를 따로 만듦
  - toList의 결과를 담지 않고 만든 리스트를 반환 -> 빈 리스트
- Mapper를 컨트롤러 계층으로 위임하면서 toEntity를 사용하지 않게 됨.
  - service에서 사용하던 매퍼를 제거하면서 entityMapper는 toDto만 가지도록 변경
- Transactional 수정
  - Transactional이 체크예외를 해결하지 못함 -> 커스텀 예외를 대부분 발생하는 현 로직에서 커스텀 예외는 Exception.class를 상속받아서 처리함.
  - rollbackFor 속성을 추가하여 BusinessLogicException.class를 처리하도록 변경.
  - 그 외에도 readOnly를 추가하여 성능 향상
- 기존에 findAll()을 사용하던 로직 전부 수정
- 그 외에도 필요없어진 코드 및 클래스 삭제 등 코드 리팩토링

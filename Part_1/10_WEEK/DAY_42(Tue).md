# DAY_42

## 프로젝트 진행사항
- JPA 어노테이션 추가
- Web 연결 후 테스트 진행 하면서 코드 리팩토링
- 페이징 적용
- UUID로 들어오는 값이 많아 UUID를 매퍼에서 처리하기 위해 map[Entity]를 추가
- mapper 내 mapEntity 사용 중 문제 발생
  - 콘솔창에 표시되는 id값과 db에 저장되는 id값이 다름.
  - setId를 사용하는 것이 문제로 판단
  - mapEntity 삭제 후 서비스계층에서 객체로 만들어서 전달
 
- @Mapping 어노테이션을 사용하지 않아 readStatus에 channel 값이 안들어감.
- online을 boolean값으로 가지고 있던걸 메서드로 시간을 기준으로 boolean을 넘겨주도록 변경 (online 필드 삭제)
- findTopByChannelOrderByUpdatedAtDesc로 최근 메시지 시각을 받도록 수정

# DAY_49

## 프로젝트 진행사항
- EmployeeLogEvnet 추가 및 구현
  - ChangeLog를 Event로 처리하기 위해 구현
- Employee Delete 추가 구현
  - EmployeeLogEvent 추가
- Employee Update 구현
  - EmployeeLogEvent 추가
- Employee Read 상세 조회 구현
- Employee ReadAll 전체 목록 조회 구현
  - 쿼리 파라미터가 많아서 Dto로 한번에 받아서 처리하도록 Dto 추가.
  - 동적 쿼리를 위해 Specitication 추가 및 구현
  - 커서 페이지네이션을 Specitication으로 구현하기 위해 greaterThan, lessThan 사용

# DAY_43

## 프로젝트 진행사항
- 메시지 생성 시 READSTATUS가 중복 생성 되는 문제 발생
  - 이미 존재하는 readStatus를 읽지 못하고 새로 생성해서 생기는 오류
  - DB에는 존재, 프론트에서 읽지 못하고 백엔드로 생성 요청
  - 백엔드 검증 로직에는 Db에 존재해서 오류 리턴
  - 메시지 자체는 생성됨.
- Storage 추가로 더이상 Db에 binaryContent 데이터(bytes)를 저장하지 않도록 변경
  - 가장 큰 용량을 차지하는 부분으로 따로 관리함으로써 db에 사용되는 용량을 줄일 수 있음
 
- Mapper를 서비스계층에서 사용하다가 Controller 계층으로 위임

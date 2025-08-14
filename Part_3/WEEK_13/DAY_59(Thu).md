# DAY 59

## 프로젝트 진행사항
- 통합 테스트 작성 완료
- 코드 리팩토링
  - Exception의 경우 when -> then을 when & then으로 통합
- issue
  - 통합테스트를 하나씩 테스트 시 문제없이 통과
    - 클래스, 패키지 단위로 한번에 테스트 시 db가 초기화 되지 않음
    - AfterTransaction 어노테이션을 사용해 repository.deleteAll() 실행
    - +UserRepository.deleteAll() 시 ReadStatus가 존재해서 삭제 안되는 상황 발생
    - 추가로 readStatusRepository.deleteAll() 추가
- 참고
  - https://sosimhan-dev.tistory.com/3
  - https://jojoldu.tistory.com/761

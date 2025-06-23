# DAY_22

## 프로젝트 진행
+ 코드 리팩토링
+ application.yaml 수정
  + 환경설정 추가
  + 환경설정을 토대로 JCF와 File을 결정할 수 있도록 함
    + repositoryConfig 클래스를 통해 @Value로 타입을 문자열로 받고 조건문을 통해 Repository 생성 / default는 jcf
    + 클래스는 @Configuration, 각 메서드에 @Bean으로 Bean 등록
    + fileRepository 내 @Value를 통해 저장될 위치를 문자열로 저장
    + Test에 더미 이미지 추가



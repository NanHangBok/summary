# DAY_68

## 프로젝트 진행사항
- S3 연동 완료
  - 테스트 코드 작성 완료
  - BinaryContentStorage 고도화 완료
- AWS RDS 연동 완료
  - SSH 터널링 테스트 완료
  - 인바운드 규칙 편집 완료
  - dataGrip 테스트 완료
- ECR 구성 완료
  - IAM 권한 부여완료
  - Docker 이미지 빌드후 배포 완료
- ECS 구성 완료
  - discodeit.env 파일 S3 업로드 완료
  - 클러스터 생성 완료
  - 작업 정의 생성 완료
  - 서비스 생성 완료
  - 인바운드 규칙 편집 완료
  - 접속 및 동작 테스트 완료
- 이미지 멀티 스테이지 최적화 완료
- CI/CD 파이프라인 구축 완료
  - test.yml 파일 생성 완료
  - CodeCov 테스트 커버리지 뱃지 추가 완료
  - deploy.yml 생성 완료
  - GitHub repo secret 추가 완료
  - GitHub repo variable 추가 완료
  - 이미지 빌드 및 푸시 job 정의 완료
  - ecs 서비스 업데이트 job 정의 완료
    - 작업 정의 생성 job 정의 완료
    - 생성된 작업으로 서비스 업데이트 job 정의 완료
    - AWS 콘솔에서 태스크 변경 확인 완료

- 참고
  - https://choiseokwon.tistory.com/395
  - https://shanepark.tistory.com/457
  - https://stackoverflow.com/questions/60826757/accessdeniedexception-an-error-occurred-when-calling-the-registertaskdefinitio
  - https://www.0x00.kr/aws/cli/ecs-update-service-container-port
  - https://yoo11052.tistory.com/162
  - https://junhyunny.github.io/spring-boot/test-container/aws/test-container-aws-s3-integration-test/

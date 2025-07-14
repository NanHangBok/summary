# DAY_36
---
## 목차
- [이론](#이론)

## 프로젝트 진행 사항
- SwaggerConfig 클래스 추가
  - ErrorResponse 공통 응답 수정
  - application.yaml 설정 SwaggerConfig로 이관
 
## 회고
- Swagger를 처음 사용해 보는거라 어노테이션이 처음에는 익숙하지 않았지만 금방 익숙해졌다. 배우기 쉬운 것 같다.
- SwqggerConfig가 가장 어려웠지만 어노테이션을 이해하고 Config를 보니 어노테이션으로 설정한 값을 따로 설정하는 것이라 한 번 이해하니 적응되었다.
  - Operation 어노테이션이나 Schema 어노테이션으로 작성한 내용을 Schema나 readOperation으로 값을 읽고 ApiResponse 어노테이션 내용을 addApiResponse로 추가하는 것이 금방 이해가 되었다.
  - 이제 내용이 무슨 내용인지는 해석할 수 있지만 아직 작성하는 법이 익숙치 않다.
- API 스펙에 맞추어서 현재 API를 수정하는 것이 까다로웠다.
  - 특히 Dto 필드(파라미터) 명이 다를 경우 DI 되지 않는 것을 찾는 것이 까다로웠다.
  - 기존에는 ModelAttribute로 파일과 Dto를 동시에 입력받고 있었는데 이것을 RequestPart로 수정하는 부분에서 MultipartFile이 입력되지 않는 상황이 굉장히 많은 시간을 소요한 것 같다.
    - 매핑은 Form-Data로 매핑을 받고 Dto는 어노테이션을 사용하여 새롭게 정의해서 JSON으로 입력 받도록 하는 과정이 많은 시간을 소요시켰다.
    - 커스텀Exception을 추가하였고 ExceptionCode를 추가하여 로깅을 좀 더 쉽게 하도록 수정하였다.

---

# 이론

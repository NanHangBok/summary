# DAY_65

## 프로젝트 진행사항
- 이슈
- org.mockito.exceptions.misusing.UnnecessaryStubbingException 발생
  - 참고 사이트 : https://widian.github.io/java/2021/08/15/mockito-%EC%82%AC%EC%9A%A9-%EC%A4%91-Unnecessary-Stubbing-Exception-%ED%95%B4%EC%86%8C%ED%95%98%EA%B8%B0.html
  - 테스트코드의 엄격성은 다음 요소들을 보장해 줍니다.
    - 테스트코드에서 사용되지 않는 stub (when/thenReturn)을 줄여줍니다.
    - 테스트코드에서 불필요한 코드 중복을 없애주고 이를 통해 필요없는 테스트 코드 역시 줄여줍니다.
    - 죽은 코드를 제거하면서 생기는 불필요한 테스트를 없애도록 도와줍니다.
    - 이를 통해 디버깅 편의성과 생산 효율을 올려줍니다.
  - -> 여기서 RequestBody.fromInputStream(multipartFile.getInputStream(), multipartFile.getSize())를 테스트코드 given과 실제 사용 두 군데서 사용하니 발생하는 오류로 확인됨.
  - -> RequestBody inputStream = RequestBody.fromInputStream(multipartFile.getInputStream(), multipartFile.getSize());
  - -> 변수로 설정 후 동일한 변수를 통해 코드 진행 후 오류 해결.
- ```
      @Test
    void download() throws Exception {
        // givne
        UUID uuid = UUID.randomUUID();
        BinaryContentDto binaryContentDto = new BinaryContentDto(uuid, 1L, "filename", "image/jpeg");
        given(presignedGetObjectRequest.url()).willReturn(new URL("https://amazon.com/test-url"));
        given(presigner.presignGetObject(any(GetObjectPresignRequest.class))).willReturn(presignedGetObjectRequest);
        // when
        ResponseEntity response = s3BinaryContentStorage.download(binaryContentDto);

        // then
        assertEquals(response.getStatusCode(), HttpStatus.FOUND);
    }
  ```
  - presignedGetObjectRequest를 given 추가하지 않아서 null 오류 발생
  - ```
        @Mock
    private PresignedGetObjectRequest presignedGetObjectRequest;
    ...
            given(presignedGetObjectRequest.url()).willReturn(new URL("https://amazon.com/test-url"));
        given(presigner.presignGetObject(any(GetObjectPresignRequest.class))).willReturn(presignedGetObjectRequest);
    ```
    - 추가로 해결

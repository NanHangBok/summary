# DAY 54

## 목차
- [예외처리 계층구조](#예외처리-계층구조)
- [통합 예외 처리](#통합-예외-처리)
- [비즈니스 예외 처리](#비즈니스-예외-처리)
- [예외 전환 전략](#예외-전환-전략)
- [로깅](#로깅)
- [Bean Validation](#bean-validation)
- [모니터링](#모니터링)
- [테스트](#테스트)

## 예외처리 계층 구조
#### 도메인 별 분리
모든 예외를 하나의 파일에 작성하면 관리가 어렵고 협업 시 충돌 가능성이 커짐
- -> 도메인 별로 예외 분리
- baseException을 상속받아서 각 도메인 별로 예외 처리 분리

#### 에러 코드 체계화 전략
- 필요성
  - 에러 상황을 빠르게 정확하게 식별하기 위해서는 텍스트 메시지만으로는 부족
  - 코드화된 에러 식별자를 사용하면 다양한 일관되게 대응할 수 있도록

## 통합 예외 처리
- 필요성
  - 컨트롤러마다 중복된 try-catch 제거
  - 예외 응답 메시지 포맷 통일
  - 예외 발생 시 공통 로깅 및 대응 가능

- @ExceptionHandler 활용
  - 개별 컨트롤러 예외 처리

- @RestControllerAdvice 활용
  - 전역 예외 처리기

## 비즈니스 예외 처리
- 도메인 규칙이나 비즈니스 규칙 위반 상황
- 시스템적인 문제가 아닌 업무 로직 상의 예외
- 필요성 (문제점)
  - 어떤 예외인지 메시지로만 구분해야 하므로 코드 레벨에서 명확성이 떨어짐
  - 예외의 의도를 파악하기 어려워 디버깅/로그 분석 어려움
  - 예외별로 다른 처리를 적용하기 힘듦
  - API 클라이언트에게 일관된 에러 응답을 제공하기 어려움
- 이점
  - 예외의 의도를 명확하게 표현할 수 있음
  - 예외마다 별도의 에러 코드와 메시지를 부여할 수 있음
  - 전역 예외 처리기에서 일관된 응답 형태로 처리 가능
 
#### 예외 발생 시점와 위치 선정
- 발생 위치
  - Controller 사용자 요청 검증, 응답 조립
  - Service 도메인 로직 조합, 트랜잭션 관리
  - Domain 도메인 규칙 보장
  - Repository 데이터 접근
- 도메인에서 예외 발생 시키는 이유
  - 도메인은 상태 불변셩을 스스로 책임 져야 함
  - 서비스에서 모든 상태 체크를 할 경우 도메인 규칙이 분산되어 유지보수가 어려워짐
  - 테스트 시 도메인 단위의 단위테스트가 가능해짐
- 도메인 예외 예시
  - 이미 탈퇴한 회원이 다시 탈퇴를 시도할 때
  - 잔액이 부족할 때 출금 요청이 들어온 겨웅
  - 이미 배송된 주문에 대해 취소 요청이 들어온 경우

- 컨트롤러 계층 예외
  - 특징
    - HTTP 상태 코드와 함께 오류 응답 반환
    - @ExceptionHandler 와 @RestControllerAdvice 활용 예외 처리
    - 입력 값 검증 관련 예외 처리
    - 클라이언트가 이해할 수 있는 오류 메시지 제공
   
- 서비스 계층 예외
  - 도메인 로직 조합 & 트랜잭션 단위 비즈니스 로직 처리
  - 하위 계층에서 발생하는 예외를 래핑하거나, 도메인 상태가 유효하지 않은 경우 직접 예외 발생
 
- 레포지토리 계층 예외 처리
  - 특징
    - 대부분의 예외를 자동으로 Spring의 DataAccessException으로 변환
    - 기본 제공 Repository 메서드는 예외 처리가 이미 되어 있음
    - 커스텀 Repository 구현 시 특정 예외 상황에 대한 직접적인 처리가 필요
    - EntityManager를 직접 사용할 때 JPA 관련 예외 처리 필요
   
  - 직접 처리 예외
    - NoResultException : 단일 결과를 기대하는 쿼리에서 결과가 없을 때
    - NonUniqueResultException : 단일 결과를 기대하는 쿼리에서 여러 결과가 반환될 때
    - PessimisticLockException : 비관적 락 획득 실패 시 발생
    - QueryTimeoutException : 쿼리 실행 시간이 초과될 때 발생
    - LockTimeoutException : 락 획득 대기 시간이 초과될 때 발생
    - PersistenceException : JPA 관련 일반적인 예외의 상위 클래스
   
## 예외 전환 전략
- 낮은 수준 예외 전환 흐름
  - ```
    // 파일 저장소 구현 (예: 파일에서 사용자 정보 로드)
    public class FileUserRepository {
        public User load(String userId) {
            try {
                // 파일에서 사용자 정보 로드
                return loadFromFile(userId);
            } catch (IOException e) {
                // 시스템 예외를 비즈니스 예외로 전환
                throw new DataAccessException("파일을 읽을 수 없습니다.", e);
            }
        }
    }
    ```
  - ```
    // 서비스 계층에서 비즈니스 예외로 한 번 더 추상화
    public class UserService {
        public UserDto getUser(String id) {
            try {
                User user = userRepository.load(id);
                return convertToDto(user);
            } catch (DataAccessException e) {
                // 내부 처리 실패를 클라이언트가 이해할 수 있도록 추상화
                throw new BusinessException(ErrorCode.DATA_ACCESS_ERROR);
            }
        }
    }
    ```
  - 필요성
    - 기술 독립성
    - 의미 명확화
    - 클라이언트 응답 일관성
    - 테스트 용이성 (IOException "파일 오류"라는 시스템적 오류 -> BusinessException "사용자 조회 실패"와 같은 비즈니스 의미 제공)
   
  - 주의 사항
    - 원본 예외 포함(cause) 필수
    - 추상화 레이어를 지킴
    - 예외 메시지 구조화

## 로깅
- 애플리케이션이 실행되는 동안 발생하는 다양한 정보를 기록해서 저장하는 행위
- 필요성
  - 애플리케이션 동적 상황 파악
  - 문제 상황 분석
  - 비즈니스 이벤트 추적
  - 보안 검사 (검사 기능)
- 주의사항
  - 민감정보 기록 금지, 레벨 구분

#### 로깅 프레임워크
- SLF4J (Simple Logging Facade for Java)
  - 다양한 로깅 프레임워크에 대한 추상화 계층
  - 실제 로그 출력은 구현체(Logback 등)가 담당
- Logback
  - 특징
    - 빠른 처리 속도와 낮은 메모리 사용
    - 설정 파일 자동 감지 및 재로딩 지원
    - 시간 기반 또는 용량 기반의 로그 파일 롤링 기능
    - 조건부 필터링 및 다양한 출력 패턴 제공
  - Spring Boot는 기본적으로 spring-boot-starter-logging에 의해 Logback을 사용
  - ```
    logging:
      level:
        root: INFO
        com.example.myapp: DEBUG
      file:
        name: myapp.log
      pattern:
        console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    ```
- 로그 정책
  - TRACE
  - DEBUG
  - INFO
  - WARN
  - ERROR
 
- 로그 출력 패턴
  - %d 날짜/시간
  - %thread 스레드 이름
  - %-5level 로그 레벨 (5자 고정)
  - %logger{36} 로거 이름 최대 36자
  - %msg 메시지 내용
  - %n 줄바꿈

- 롤링 정책
  - 로그 파일이 너무 커지거나 오래 저장되지 않도록 자동으로 분할하고 오래된 로그를 삭제하거나 압축하는 관리 전략
  - 필요성
    - 로그 파일이 계속 한 파일에 쌓이면 용량이 매우 커질 수 있음
    - 이로 인해 디스크 용량을 고갈시키거나, 로그 열람 속도가 느려질 수 있음
    - 운영 환경에서는 오래된 로그를 일정 기간만 보관하고 자동 삭제하는 것이 일반적임
   
  - 롤링 정책의 주요 유형
    - 시간 기반 롤링
    - 크기 기반 롤링
    - 시간 + 크기 기반

#### 환경 별 로깅 전략
- @Profile이나 spring.profiles.active에 따라 환경을 구분하고 Logback 설정 파일에서 <springProfile>을 사용하여 환경별 설정을 나눌 수 있다.
- 로그 관리 전략
  - maxHistory 로그 보관 일 수 설정
  - totalSizeCap 로그 전체 크기 제한. 초과 시 오래된 로그부터 삭제
  - fileNamePattern 로그 파일 이름에 날짜를 포함시켜 일별 분할 설정
  - cleanHistoryOnStart 애플리케이션 시작 시 오래된 로그를 즉시 삭제
 
- 로그 레벨 조정
  - 실행 이후에 로그 레벨을 동적으로 조정 : Spring Boot Actuator의 loggers 엔드포인트를 사용하여 가능.
  - ```
    management.endpoints.web.exposure.include=loggers
    management.endpoint.loggers.enabled=true
    ```
  - ```
    curl -X POST -H "Content-Type: application/json" \\
         -d '{"configuredLevel": "DEBUG"}' \\
         <http://localhost:8080/actuator/loggers/com.example.myapp>
    ```
- 환경 별 로그 레벨
  - Dev :DEBUG
  - test : INFO
  - prod : WARN


#### 로그 레벨별 활용 전략
- TRACE : 반복문 내부 흐름, DB 쿼리 상세 추적 등
- DEBUG : 서비스 간 호출 흐름, 주요 변수 상태 출력
- INFO : API 진입, 처리 완료, 주요 이벤트 기록
- WARN : 리트라이 발생, 한계 접근, 입력 이상치 감지
- ERROR : 예외 발생, 기능 실패

#### 로그 메시지 작성 규칙
- 충분한 컨텍스트 정보 포함
  - 누가, 무엇을, 어떤 상태에서, 어떻게 변했는지, 수행했는지를 함께 기록
- 일관된 형식 유지
  - 검색, 필터링, 모니터링이 훨씬 쉬워짐 ( key=value 형태 )
  - 로그 형식을 사전에 정의
- 검색 가능한 키워드 사용
  - 정형화된 키워드 또는 태그를 삽입하면, 특정 작업 단위, 기능, 에러 유형을 빠르게 검색할 수 있음
- 파라미터화된 메시지 사용
  - 문자열 연결 (+) 대신 {} 형태의 파라미터 바인딩 방식 지원
  - ```
    logger.debug("사용자 ID: " + userId + ", 이름: " + userName);  // X
    logger.debug("사용자 ID: {}, 이름: {}", userId, userName);  // O
    ```
  - 현재 로깅 레벨에 따라 실제 문자열을 조합할 지 말지를 판단하므로 성능이 향상됨
  - // 문자열이 항상 연결되기 때문에 로그 출력 조건이 DEBUG가 아니더라도 연산은 수행됩니다.
- 민감한 정보 제외

#### 예외 상황 로깅 전략
- 반드시 예외 객체(e)를 로그의 마지막 파라미터로 포함해야 스택 트레이스가 출력 됨.
- 예외 수준에 맞는 로그 레벨 선택
  - 모든 예외 상황을 무조건 ERROR로 처리하는 것은 과도한 알림을 발생시키고 경고의 의미를 약화시킬 수 있다.
  - 예외의 심각도와 복구 가능성에 따라 WARN, ERROR 등의 로그 레벨을 적절히 분리해야 함
 
- 예외 컨텍스트 정보 포함
  - 단순히 예외 메시지나 스택 트레이스만 출력하면 **문제 발생 원인을 파악하기 어렵다**
  - 예외가 발생한 시점의 주요 상태 정보(컨텍스트)를 함께 로그에 남기면 문제 해결 속도가 향상 됨

#### 로그 분석과 활용
- 주요 로깅 포인트
  - 애플리케이션 시작과 종료
  - 요청 처리 시작과 종료
  - 중요 비즈니스 로직
    - 도메인 핵심 로직에서 적절한 시점 마다
  - 외부 시스템 연동
    - DB 조회, API 호출, Redis 접근 등 외부 리소스에 대한 성능 병목 파악을 위해
  - 로깅과 성능 영향
    - 루프 내부에서 로그를 과도하게 출력하는 것은 오히려 성능 저하를 유발할 수 있음.

#### 로그 검색과 분석
- 문제 발생 시 빠르게 원인을 찾기 위해
- 검색 필요 상황
  - 장애 발생 시 원인 확인
  - 사용자 민원 발생 시 로그 추적
  - 느린 응답의 병목 위치 파악
    - 병목만 고려할 것이 아니라 호출 횟수도 같이 고려할 것.
    - 드물게 호출되는 병목 보단 자주 호출되는 응답 시간을 줄이는 것.
  - 외부 API 오류 확인
  - 예상치 못한 예외 발생 시 재현 조건 파악
- 기본적 로그 검색
```
# 키워드 기반 검색
$ grep "ERROR" app.log
$ grep "[USER_CREATE]" app.log

# 시간 범위 지정 (로그가 타임스탬프 포함 시)
$ awk '/2024-12-01 10:00/,/2024-12-01 11:00/' app.log

# 특정 사용자 ID 로그 찾기
$ grep "userId=123" app.log
```
- 정규 표현식 활용
```
# 예: userId가 숫자인 로그만 추출
$ grep -E "userId=[0-9]+" app.log

# 예: 특정 모듈에서 발생한 WARN 이상 로그 찾기
$ grep -E "\\[(USER|ORDER)_.*\\]" app.log | grep -E "WARN|ERROR"
```

#### 문제해결 활용
- 느린 응답 원인 추적
- 사용자 민원 추적
- 예외 재현 조건 파악

- 로그에는 가능한 한 사용자 ID, 요청 ID, 트랜잭션 ID를 포함하는 것이 좋다.

## Bean Validation
- 데이터 검증 필요성
  - 잘못된 데이터 유입으로 인한 시스템 오류
  - -> 일관되지 않은 검증 로직 분산 문제
  - -> 비즈니스 규칙 위반 방지
  - -> 사용자에게 명확한 피드백 제공

- 제공 기능
  - Controller 메서드 파라미터 자동 검증
  - @Valid(Dto), @Validated(클래스) 어노테이션을 통한 선언적 방식 지원
  - 검증 실패 시 예외 처리 메커니즘 제공
  - 사용자 정의 검증기 등록 및 확장 가능
  - `implementation 'org.springframework.boot:spring-boot-starter-validation'`
    - Spring Boot 2.3.0 이전 버전에서는 spring-boot-starter-web 안에 validation 의존성이 포함되어 있었지만, 이후 버전부터는 별도로 추가해야 합니다.

```java
import jakarta.validation.constraints.*;

public class User {

    @NotBlank(message = "사용자 이름은 필수입니다.")
    private String username;

    @Email(message = "이메일 형식이 올바르지 않습니다.")
    private String email;

    @Min(value = 18, message = "나이는 18세 이상이어야 합니다.")
    private int age;

    public User(String username, String email, int age) {
        this.username = username;
        this.email = email;
        this.age = age;
    }
}
// @Validated
```
- Bean Validation의 기본 제약 조건으로 @NotNull, @Size, @Email, @Min, @Max, @Pattern 등을 제공

#### 주요 검증 어노테이션
- @NotNull null불허, @NotEmpty null과 ""불허, @NotBlank null,"",공백(" ") 불허
- @Size(문자열, 배열, 리스트 등 길이/크기), @Length(문자열 길이 전용/hibernate 전용) `@Size(min=10,max=50)`
- @Email(이메일 주소 형식), @Pattern(정규식을 이용한 형식 검증) `@Pattern(regexp ="^\\d{3}$")`
- @Min, @Max, @Range
- @Positive, @PositiveOrZero, @Negative, @NegativeOrZero
- @Digits, @DecimalMin, @DecimalMax
- @Past, @Feture, @PastOrPresent, @FutureOrPresent
- 기본적으로 컬렉션 중첩 객체의 검증은 불가 -> @Valid 어노테이션 추가 필수
  - `private List<@NotNull @Valid Item> validItems;`
  - ```
    @Valid
    private List<Item> validItems;
    ```

#### 검증 수행 위치
- Controller
  - 컨트롤러 메서드의 파라미터에 @Valid 또는 @Validated를 적용
  - 검증 실패 시 BindingResult를 통해 오류 정보 확인
  - @Valid = Bean 검증
  - 메서드 파라미터 검증 = 클래스에 @Validated 사용, @NotNull 등 어노테이션 사용
- Service
  - 장점
    - 컨트롤러와 분리되어 테스트 용이
    - 복잡한 비즈니스 검증을 재사용 가능
    - 도메인 중심의 책임 분리가 가능
- 검증 그룹 활용
  - ```java
    // 마커 인터페이스 정의
    public interface OnCreate {}
    public interface OnUpdate {}

    // 검증 대상 객체
    public class UserForm {
  
      @NotNull(groups = {OnCreate.class, OnUpdate.class})
      private String name;
  
      @NotNull(groups = OnCreate.class)
      @Size(min = 8, groups = OnCreate.class)
      private String password;
  
      @NotNull(groups = OnCreate.class)
      @Email(groups = {OnCreate.class, OnUpdate.class})
      private String email;
  
      @Null(groups = OnCreate.class)
      @NotNull(groups = OnUpdate.class)
      private Long id;
    }
    // 그룹별 검증 실행
    public class UserForm {

      @NotNull(groups = {OnCreate.class, OnUpdate.class})
      private String name;
  
      @NotNull(groups = OnCreate.class)
      @Size(min = 8, groups = OnCreate.class)
      private String password;
  
      @NotNull(groups = OnCreate.class)
      @Email(groups = {OnCreate.class, OnUpdate.class})
      private String email;
  
      @Null(groups = OnCreate.class)
      @NotNull(groups = OnUpdate.class)
      private Long id;
    }
    ```

#### 커스텀 Validator
- 구현절차
  - Custom Annotation 정의 (@interface)
  - ConstraintValidator 인터페이스를 구현하여 검증 로직 작성
  - DTO 클래스의 필드에 Custom Annotation 적용
  - `@Constraint(validatedBy = {NotSpaceValidator.class}) // 검증 로직을 수행할 Validator 클래스를 지정`
  - initialize() {} // 초기화 로직
  - boolean isValid() {}
 
- 정규 표현식 vs Custom Validator
  - 정규 표현식은 간결하고 재사용 가능하지만 성능 저하 우려, 복잡함
  - Custom validator 기반 검증은 복잡한 로직 가능, 가독성 우수 but 구현 필요, 다소 장황함
 
#### 복합 객체 검증
- 중첩된 객체 검증
  - 중첩 객체에 추가해야 하위 객체의 제약 조건이 검증 됨.
  - @Valid 애너테이션을 중첩 객체 필드에 명시하면 객체 내부 필드에 대한 검증까지 수행됨
  - 생략 시 내부 필드 검증이 무시됨.
 
- 컬렉션 요소 검증
  - @Valid는 컬렉션 타입에 대해서도 요소 하나하나 검증 전파
  - 컬렉션 전체에 대한 조건은 @Size를 통해 적용 가능
 
- 상황별 검증 전략
  - Controller : @Valid @RequestBody, @ModelAttribute / 기본적인 필드 검증, 단순 구조에 적합
  - Service : @Validated 클래스 레벨 적용 / 복합 객체, 비즈니스 로직 중심의 검증에 적합
  - Nested DTO : @Valid 필드 적용 / 중첩 객체 및 컬렉션 요소 검증 가능

## 모니터링
- 실시간 상태 모니터링
  - 시스템 다운타임 최소화
  - 사용자 경험 향상
  - 운영 효율성 증가
  - 장애 대응 시간 단축
 
- 잠재적 무기 조기 발견

- 주요 지표 수집과 분석
  - 시스템 자원 : CPU, Memory, Disk 등
  - 접근 및 성능 : 응답 시간, 처리량, 오류율 등
  - 데이터베이스 : 쿼리 실행 시간, 커넥션 풀 상태
  - 외부 시스템 : API 호출 성공률, 응답 시간 등
  - 이러한 지표는 모니터링 대시보드로 활용 됨
 
- 정상 동작 여부 확인 (Health Check)
  - 서비스가 정상적으로 동작하고 있는지 주기적으로 확인하는 것은 안전성 확보에 있어 매우 중요
  - 이를 자동으로 처리해주는 것이 Spring Boot Actuator

#### 사용자 정의 HealthIndicator
```
// ExternalApiHealthIndicator.java
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    private final RestTemplate restTemplate;
    private final String apiUrl;

    public ExternalApiHealthIndicator(
        RestTemplate restTemplate,
        @Value("${external.api.url}") String apiUrl) {
        this.restTemplate = restTemplate;
        this.apiUrl = apiUrl;
    }

    @Override
    public Health health() {
        try {
            // 외부 API에 /health 엔드포인트 요청
            ResponseEntity<String> response = restTemplate.getForEntity(apiUrl + "/health", String.class);

            if (response.getStatusCode().is2xxSuccessful()) {
                // 2xx 응답이면 UP 상태로 간주
                return Health.up()
                        .withDetail("statusCode", response.getStatusCodeValue())
                        .withDetail("message", "External API is available")
                        .build();
            } else {
                // 비정상 응답 처리
                return Health.down()
                        .withDetail("statusCode", response.getStatusCodeValue())
                        .withDetail("message", "External API returned non-2xx status")
                        .build();
            }
        } catch (Exception e) {
            // 예외 발생 시 DOWN 상태로 간주
            return Health.down()
                    .withDetail("message", "External API is not available")
                    .withDetail("error", e.getMessage())
                    .build();
        }
    }
}
```
- DB 헬스 쿼리 커스터마이징 가능
```
# DB 헬스 체크 활성화
management:
	health:
		db:
			enabled: true

# 커넥션 풀(HikariCP) 헬스 체크 설정
spring:
	datasource:
		hikari:
			health-check-properties:
				connectivityCheckTimeoutMs: 1000
				expected99thPercentileMs: 10
// connectivityCheckTimeoutMs: 커넥션 체크 타임아웃 (1초)
// expected99thPercentileMs: 연결 시간의 99%가 이 값을 넘지 않도록 기대 (10ms)
```
- 이름 지정 및 조건부 활성화
  - `@Component("customName")` : 인디케이터 이름을 명시적으로 지정
  - `@ConditionalOnEnabledHealthIndicator("customName")` : 설정 값에 따라 인디케이터를 조건부로 등록
 
- Mono<> : Spring Webflux: 비동기 논블로킹

#### Health Group 설정
- Health Indicator를 그룹으로 구성 가능.
- 운영 목적에 따라 헬스 응답을 분리하여 더 유연하게 시스템 상태를 점검 가능

#### 주요 엔드포인트 활용
- metrics / 시스템 성능 지표
  - `http://localhost:8080/actuator/metrics` 기본 호출
  - `http://localhost:8080/actuator/metrics/jvm.memory.used` 특정 지표 조회
- 런타임 로그 레벨 조정
  - `/actuator/loggers` 실행중인 애플리케이션의 로그 레벨을 동적으로 조회 및 변경 가능.
  - `GET http://localhost:8080/actuator/loggers` 전체 로그 조회
  - `GET http://localhost:8080/actuator/loggers/com.example.my.app` 특정 로그 조회
  - ```
    // 실행중인 로그 레벨 동적 변경
    POST http://localhost:8080/actuator/loggers/com.example.myapp
    Content-Type: application/json
    
    {
      "configuredLevel": "WARN"
    }
    ```
- info
  - `/actuator/info` 애플리케이션 관련 메타데이터를 외부에 제공하는 용도
  - ```
    management:
      info:
        env:
          enabled: true   # ✅ 추가!
    info:
      app:
        name: Spring Actuator Application
        description: Spring Boot Actuator Practice Application
        version: 1.0.0
      team:
        name: Codeit Backend Team
        contact: backend@codeit.com
    ```
  - git 정보 연동 가능
    - git 커밋 정보, 브랜치 정보 등 버전 제어 시스템의 상태를 자동으로 포함할 수 있음.

## 테스트
어떤 대상이 정해진 기준에 부합하는지를 검증하는 과정
- 장점
  - 코드 품질 보장 : 기능이 정확히 동작하는지
  - 리팩토링 용이 : 테스트가 있으면 내부 코드를 개선할 때 안심하고 수정할 수 있음
  - 회귀 테스트 가능 : 기존에 되던 기능이 수정 이후에도 잘 동작하는지 자동으로 확인 가능
  - 문서화 역할 : 테스트 코드 자체가 예제이자 문서의 역할
  - 협업 효율 향상 : 코드 변경 시, 사전에 문제가 발생할 부분을 빠르게 인지할 수 있음
 
- 수동 테스트 한계
  - 매번 서버를 실행하고 Postman 띄우는 과정이 번거로움
  - 테스트 항목이 많아질수록 반복 작업이 과중
  - 일부 기능은 단독 테스트가 어려움
 
- 단위 테스트
  - 작은 단위의 기능이나 메서드가 기대한 결과를 정확히 내는지 테스트하는 것
  - 애플리케이션 전체를 실행하지 않고
  - 빠르게 반복 수행할 수 있으며
  - 자동화하기 쉽다.
  - JUnit, AssertJ 등 사용해 작성
  - Spring은 계층별 테스트(@WebMvcTest, @DataJpaTest 등)를 지원
 
- 테스트 필요 상황
  - 신규 기능 개발 시
  - 버그 수정 시
  - 리팩토링 시
  - 협업 중 기능 변경 시
  - 외부 API 연동 시
 
#### 테스트 종류
- 단위 테스트 : 메서드의 기능 검증
- 통합 테스트 : 여러 컴포넌트 간의 연동 검증
- 슬라이스 테스트 : 특정 계층만 검증

- 단위 테스트 FIRST
  - Fast 빠르게
  - Independent 독립적으로
  - Repeatable 반복 가능하도록
  - Self-validating 셀프 검증이 되도록
  - Timely 시기적절하게
 
- 기능 테스트
  - 애플리케이션을 사용하는 사용자 입장에서 애플리케이션이 제공하는 기능이 올바르게 동작하는지를 테스트
 
- 통합 테스트
  - 애플리케이션을 만든 개발자 또는 개발팀이 테스트의 주체
  - 클라이언트 측 툴 없이 개발자가 짜 놓은 테스트 코드를 실행시켜서 이루어지는 경우가 많음
  - MockMvc를 사용하여 HTTP 요청을 실제처럼 보냄
  - 컨트롤러, 서비스, 리포지토리 계층이 실제로 동작
  - 장단점
    - 실제 환경과 유사, 속도가 느림
    - 전체 흐름 검증 가능, 디버깅 난이도 높음
   
- 슬라이스 테스트
  - 특정 계층으로 쪼개어서 하는 테스트
  - Mock 객체를 사용해서 계층별로 끊어서 테스트할 수 있기 때문에 어느 정도 테스트 범위를 좁히는 것이 가능
  - 단위 테스트라고 부르기에는 단위가 큰 테스트이며, 애플리케이션의 일부만 테스트하기 때문에 부분 통합 테스트라고 부르기도 한다.


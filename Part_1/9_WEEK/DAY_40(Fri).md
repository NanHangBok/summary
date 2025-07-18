# DAY_40

---
## 목차
- [선언적 트랜잭션](#선언적-트랜잭션)
- [프로그래밍 방식 트랜잭션](#프로그래밍-방식-트랜잭션)
- [AOP와 트랜잭션](#aop와-트랜잭션)
- [트랜잭션 매니저](#트랜잭션-매니저)
- [트랜잭션 전파](#트랜잭션-전파)
- [트랜잭션 격리 수준](#트랜잭션-격리-수준)
- [트랜잭션 롤백](#트랜잭션-롤백)
- [성능 최적화](#성능-최적화)
- [트랜잭션 문제](#트랜잭션-문제)
  - [내부 메소드 호출 문제](#내부-메소드-호출-문제)

---

## 선언적 트랜잭션
- 스프링에서는 AOP 기반으로 트랜잭션을 관리하며, 가장 많이 사용하는 방법은 @Transactional 어노테이션을 선언하는 방식
- 클래스 레벨에 적용 시 모든 **public** 메서드에 적용
- 메서드 레벨에 적용 시 해당 설정이 적용됨.
- 주요 속성
  - propagation : 전파 방식
  - isolation : 격리 수준
  - timeout : 제한 시간(초) / -1(무제한)
  - readOnly : 읽기 전용 여부 / 조회 전용
  - rollbackFor : 롤백 발생 예외
  - noRollbackFor : 롤백 제외 예외
- 비즈니스 로직과 분리/ 코드 간결 -> 복잡한 흐름 제어 어려움
- 유지보수 용이 / 표준적 사용 -> 내부 메서드 호출, PRIVATE 메서드 적용 안됨

## 프로그래밍 방식 트랜잭션
- 템플릿 방식
  - SPRING이 제공하는 프로그래밍 방식 트랜잭션 관리
  - 람다, 콜백 기반으로 트랜잭션 범위를 명확히 지정 가능
  - rollback : status.setRollbackOnly() 수동 지정
  - 사용 조건 : 복잡한 흐름, 조건부 rollback 시 유용
  - transactionTemplate.execute(stats -> { try {status.setRollbackOnly()} catch() {}
- 명령형 수동 방식(PlatformTransactionManager/transactionManager)
  - 가장 저수준의 트랜잭션 관리 API
  - 시작/커밋/롤백을 개발자가 직접 호출
  - 복잡한 흐름, 중첩 트랜잭션, 다른 트랜잭션 속성 사용 시 필요.
  - 전파/격리 등 세부 제어 -> TransactionDefinition 사용
  - rollback = rollback(status) 명시 호출
  - 복합 트랜잭션이나 REQUIRES_NEW 필요 시 -> 전파 방식
 
- 내부호출, private 메서드는 AOP 적용 안됨

## AOP와 트랜잭션
- spring AOP = 메서드에만 적용
- 핵심 AOP 용어
  - Aspect : join point + advice / 여러 객체에 공통적으로 적용되는 기능
  - Join Point : 프로그램 실행 중 특정 시점 (ex:메서드 실행)
  - Advice : 특정 joint point에서 실행되는 코드
  - pointcut : join point의 집합을 지정하는 표현식
  - Proxy : 대상 객체를 감싸는 객체, advice를 적용하는 역할
 
- 트랜잭션 프록시
  - 실 서비스 객체를 감싸며, 메서드 호출 전후에 트랜잭션을 자동으로 관리한다.
  - 동작흐름
    - @transactional 메서드 호출
    - Proxy가 호출을 가로채 TransactionIntercepter 호출
    - PlatformTransactionManager가 트랜잭션 시작
    - 실제 대상 객체의 비즈니스 로직 실행
    - commit, rollback
   
  - ProxyFactoryBean : 메서드 호출을 가로챔 > TransactionIntercepter에 넘김
  - TransactionIntercepter : 트랜잭션 관리 로직 담당 > PlatformTransactionManager를 이용해서 동작 시킴
  - PlatformTransactionManager : 트랜잭션의 구체적인 구현체. 트랜잭션을 열고, 커밋, 롤백하는 주체
 
## 트랜잭션 매니저
- JpaTransactionManager
  - 특징
    - EntityManagerFactory 관리 : EntityManagerFactory를 통해 EntityManager를 관리
    - 트랜잭션과 바인딩 : 트랜잭션한 범위 동안 EntityManager를 스레드에 바인딩
    - 영속성 컨텍스트 관리 : 트랜잭션 단위로 영속성 컨텍스트를 유지
    - 플러시 모드 관리 : flush 모드를 설정하여 커밋 시점에 데이터 반영을 제어

## 트랜잭션 전파
- 현재 트랜잭션이 진행 중일 때, 하위 메서드가 새로운 트랜잭션을 사용할지 기존 트랜잭션에 참여할지를 결정하는 설정
- @Transactional(propagation = REQUIRED) / 현재 트랜잭션이 있으면 참여, 없으면 생성
- 서로 다른 클래스에 트랜잭션은 병합이 문제 없음 -> 동일 클래스에서 직접 호출 시 문제가 발생할 수도 있음
- 옵션
  - REQUIRED : 트랜잭션 있으면 참여, 없으면 생성
  - REQUIRES_NEW : 항상 새로운 트랜잭션 생성 / 감사 로그, 이력 저장
  - SUPPORTS : 있으면 참여, 없으면 비트랜잭션 / 조회,로깅
  - MANDATORY : 반드시 기존 트랜잭션 필요, 없으면 예외 / 상위 트랜잭션 필수 로직
  - NEVER : 기존 트랜잭션이 있으면 예외, 없으며 비트랜잭션 / 외부 API 호출
  - NOT_SUPPORTED : 트랜잭션 중단, 비트랜잭션으로 시작 / 로그 저장
  - NESTED : 중첩 트랜잭션, SavePoint 사용 / 대용량 처리 중 일부 롤백

## 트랜잭션 격리 수준
- 일관성과 동시성을 제어하는 설정
- 문제점
  - Dirty Read : 커밋되지 않은 데이터를 다른 트랜잭션이 읽는 현상
  - non-repeatable Read : 같은 쿼리를 실행했을 때 이전과 다른 결과가 나오는 현상
  - Phantom Read : 일정 범위의 레코드를 두 번 이상 읽을 때 이전에 없던 레코드가 나타나는 현상

- 격리 수준
  - READ UNCOMMITTED
    - 가장 낮은 격리 수준, 커밋되지 않은 데이터를 읽을 수 있다
    - 실시간 모니터링 등 정확성보다 속도가 중요한 경우
   
  - READ COMMITTED
    - Dirty Read 방지, non-repeatable Read, Phantom Read 발생 가능
    - 일반적인 서비스에서 많이 사용, 적절한 일관성과 성능 제공
   
  - REPEARTABLE READ
    - 트랜잭션이 시작될 때 스냅샷을 생성, 끝날 때까지 동일한 데이터만 조회
    - Dirty Read, Non-repeatable Read 방지, Phantom Read 발생 가능
    - 트랜잭션 동안 동일 데이터를 반복 조회 시 값이 유지 됨
    - 범위 커리 시 추가된 데이터가 나타날 수 있음
   
  - SERIALIZABLE
    - 가장 높은 격리 수준
    - 모든 트랜잭션을 순차 시행처럼 보이게 함
    - 모두 방지
    - 성능 저하 및 교착 상태 발생 가능, 높은 일관성 필요 시 사용
   
## 트랜잭션 롤백
- Checked Exception은 롤백 예외 -> 예상 가능한 예외라 가정하여 커밋
  - Checked Exception은 rollbackFor로 명시해야 롤백됨
- rollbackFor = 체크 예외라도 롤백 가능하게 함
- noRollbackFor = 언체크 예외라도 롤백 방지 함
- 외부 API / 네트워크 실패 = Checked Exception -> 명시적 rollbackFor 필요

## 성능 최적화
- readOnly 속성 활요
  - 스냅샷 사용 x
  - 변경 감지를 막고, flush/ Dirty Checking 생략
  - 트랜잭션 종료 시점, 스냅샷과 비교하여 변경된 상태면 UPDATE 쿼리를 자동으로 날린다.
 
- 트랜잭션 시간 최소화
  - DB Connection 오래 점유 x
  - 최소한의 로직 범위에만 트랜잭션 부여
  - 외부 API 호출, 대기 로직 트랜잭션 안에 금지
  - 필요 시 이벤트 발행(비동기)로 처리 권장

- 불필요한 트랜잭션 제거
  - 단순 조회
  - 로그 기록
  - 외부 시스템 연동 (kafka 등)

## 트랜잭션 문제
#### 내부 메소드 호출 문제
- @Transactional은 프록시 기반으로 동작
- 같은 클래스 내 메소드 직접 호출은 프록시를 거치지 않음
- 트랜잭션이 적용되지 않고, 예상한 롤백이 동작하지 않음
  - @Transactional은 프록시 객체를 만드는 어노테이션 -> 원본객체는 따로 있음
  - 프록시 객체가 super.method() 호출하면 원본객체의 메서드가 호출되기 때문에 트랜잭션 기능이 동작하지 않음
 
- 해결법
  - ~~자기 자신을 주입 / public class MemberService { private final MemberService self;... self.inner()...}~~
  - 구조 분리 : 다른 클래스로 분리
 
#### 트랜잭션 전파 실수 (Propagation)
- 상위 트랜잭션과 하위 트랜잭션이 연관이 없다면 propagation = Propagation.REQUIRES_NEW 로 하위 트랜잭션 commit.
- 상위 트랜잭션의 마지막에 예외상황이 발생 해 롤백하게 되었을 때,
  - 하위 트랜잭션의 propagation이 default라면 같이 롤백 됨.
  - 하위 트랜잭션이 로깅, 기록용이면 기록을 남길 필요가 있는데 롤백 됨. -> REQUIRES_NEW
 
#### 긴 트랜잭션 범위 문제
- 불필요하게 Connection 오래 점유
- 외부 API, 대기 로직 포함 -> 성능 저하 / 교착상태 위험
- 따로 배기

#### 예외 처리 누락으로 인한 실패
- Checked Exception은 롤백이 아니라 커밋 됨.
- rollbackFor 설정















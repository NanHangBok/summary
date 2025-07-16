# DAY_38

---
## 목차
- [물리적 모델링](#물리적-모델링)
  - [테이블 설계](#테이블-설계)
  - [KEY](#key)
  - [참조 무결성 법칙](#참조-무결성-법칙)
  - [성능 최적화](#성능-최적화)
- [역정규화](#역정규화)
- [ORM](#orm)
- [JPA](#jpa)
  - [JPA 구성요소](#jpa-구성요소)
- [Spring Data JPA](#spring-data-jpa)
  - [Entity](#entity)

---
## 물리적 모델링
#### 테이블 설계
- 테이블 명명 규칙의 필요성
  - 협업, 유지보수, 코드 가독성 향상
  - ERD, SQL, 소스 코드의 일관성 확보
- 대표 규칙 예시
  - 소문자 사용
  - 단어 구분 : 언더스코어
  - 복수/단수
  - 접두사 지양
 
- 컬럽 데이터 타입
  - 설계 원칙
    - 저장 효율 + 무결성 + 가독성 고려
    - 과도한 크기나 잘못된 타입 사용 지양
  - 주요 타입
    - VARCHAR(n) 가변 길이
    - CHAR(n) 고정 길이
    - TEXT 검색 비효율 증가
    - SMALLINT, INT, BIGINT 정수
    - DECIMAL(n,m), NUMERIC(n,m) 실수
    - DATE 날짜
    - TIMESTAMP 날짜+시간
    - BOOLEAN 불리언
    - UUID
   
- 기본값과 제약조건
  - 목적
    - 데이터 무결성 확보
    - 예측 가능한 데이터 구조 유지
  - 주요 제약조건
    - PRIMARY KEY
    - UNIQUE
    - NOT NULL
    - DEFAULT
    - FOREIGN KEY
   
#### KEY
- 기본키
  - 테이블 내 행을 유일하게 식별하는 컬럼 또는 컬럼 조합
  - NULL 불가, 중복 불가
  - 생성 전략
    - SERIAL
    - BIGSERIAL 대용량 SERIAL
    - UUID
    - 복합 키
   
- 외래키
  - 다른 테이블의 기본키를 참조하여 관계 설정
  - 데이터 무결성 보장
  - 상위 테이블과 하위 테이블 관계 명확화
 
  - 옵션
    - ON DELETE CASCADE 참조 대상 삭제 시 함께 삭제
    - ON UPDATE CASCADE 참조 대상 변경 시 함께 변경
    - RESTRICT / NO ACTION 기본값, 변경/삭제 거부
   
  - 순환 구조 피하기
  - 데이터 삭제 전략 명확히 정의
  - 대량 트랜잭션(분산 트랜잭션)에서는 FK 지양하는 경우도 있음
 
- 고유키 (UNIQUE)
  - 중복되지 않는 값을 요구하지만, null 허용 여부는 명시 가능
  - 후보키로 사용되거나, 검색 최적화 용도로 활용
  - 복수 사용 가능
  - 특징
    - null 허용
    - 중복 불가
    - 유니크 보장
    - 여러개 가능
   
#### 참조 무결성 규칙
- 관계형 DB에서 상위 테이블의 값이 없는 상태로 하위 테이블의 값이 존재하지 않게 하는 규칙
- 필요성
  - 데이터 신뢰성 보장
  - 관계 명확화
  - 유지보수 용이
 
- CASCADE
  - 부모 테이블 데이터 변경/삭제 시 자식 테이블에 동일하게 반영
  - 삭제 시 연쇄 삭제, 변경 시 연쇄 변경 발생
  - 코드 간결 : 실수 시 데이터 대량 삭제 위험
  - 일관성 유지 : 모든 연관 데이터 삭제 됨
  - 관리가 필요한 부모-자식 관계에만 사용, 중요 데이터는 신중히 검토(댓글-게시글)
 
- SET NULL
  - 부모 테이블 삭제/변경 시 자식 테이블 외래키를 NULL로 변경
  - 외래키가 NULL 허용이어야 함
  - 유연한 유지 : NULL 데이터 후속 처리 필요
  - 데이터 보존 : 데이터 의미 모호 가능
  - 관리자 ID, 옵션 항목 등 참조값 유실 가능 시 사용, 후처리 로직 필요 (탈퇴 회원의 주문)
 
- RESTRICT / NO ACTION
  - 부모 테이블 데이터 삭제/변경 불가 (참조 중이면 거부)
  - RESTRICT, NO ACTION 동일하게 작동
  - 데이터 보호 : 삭제/변경 시 명시적 확인 필요
  - 무결성 강함 : 개발 초기 복잡도 증가
  - 실무 기본값, 명시적 삭제 로직이 필요할 때 선호됨. 트랜잭션 단위에서 안전하게 처리 가능
 
#### 성능 최적화
- 인덱스
  - 데이터베이스에서 검색 속도를 빠르게 하기 위한 자료구조
  - 인덱스를 활용하지 않으면 테이블의 모든 데이터를 순차 탐색(Full Scan) 해야 함.
 
- 인덱스 종류
  - B-Tree : 가장 일반적 / 범위 검색에 강함
  - Hash : 해시 테이블 기반 인덱스 / '=' 검색만 가능
  - GiST : 공간데이터, 복합 데이터의 적합
  - GIN : 배열, JSON, Full Text Search 등에 특화
  - BRIN : 대용량, 순차적 데이터에 적합
 
- 인덱스 설계 원칙
  - 읽기 성능 우선 : 조회가 빈번한 컬럼에 인덱스 우선 적용
  - 카디널리티(중복도) 고려 : 값이 다양할수록 효과적
  - 조회 패턴 반영 : WHERE, JOIN, ORDER BY에 활용되는 컬럼 우선
  - 과도한 인덱스 지양 : 조회 이외(INSERT, UPDATE, DELETE) 성능에 부하 발생
 
- 복합 인덱스 (다중 컬럼) 가능
  - 선행 컬럼부터 필터링
  - ORDER BY가 자주 발생하는 컬럼 순서를 맞추면 효과적
  - EXPLAIN ANALYZE로 쿼리 성능 확인
 
- ```sql
  CREATE INDEX idx_members_email
  ON members(email);

  CREATE INDEX idx_default_btree ON members USING hash (email);
  ```
 
- 인덱스 옵션
  - WHERE 조건
    - 디스크 공간 절약 & 성능 극대화
    - ```sql
      CREATE INDEX idx_members_active
      ON members(created_at)
      WHERE status_cd = 'ACTIVE';
      ```

  - 표현식 기반
    - 가공된 값으로 WHERE 절 검색 시 성능 극대화 가능
    - 이메일을 항상 소문자로 WHERE 조건에 사용한다면, 날짜의 년/월로 조회가 많다면
    - ```sql
      -- 이메일을 항상 소문자로 WHERE 조건에 사용한다면
      CREATE INDEX idx_lower_email
      ON members (LOWER(email));
      
      -- 날짜의 년/월로 조회가 많다면
      CREATE INDEX idx_extract_month
      ON orders (EXTRACT(MONTH FROM order_date));
      ```
      
- 인덱스 재구성
  - ```sql
    -- REINDEX
    REINDEX INDEX idx_members_email;
    -- DROP INDEX
    DROP INDEX idx_members_email;
    CREATE INDEX idx_members_email ON members(email);
    -- CLUSTER (물리 정렬)
    CLUSTER members USING idx_members_email
    ```
  - REINDEX는 주기적으로 (특히 UPDATE가 많은 경우)
 
## 데이터 모델 검증
- 일반적인 안티패턴 사례
  - 잘못된 기본키 설정
    - 주민번호, 전화번호 사용
    - 이메일 사용
    - 이름 사용
    - SERIAL / UUID / 전용 식별자 사용
   
  - 잘못된 일대다 관계 설정
  - 순환 참조 구조
 
- 문제점
  - 삭제/삽입 시 FK 무결성 문제 발생
  - 조인 복잡도 증가, 관리 어려움
 
- 정규화 규칙 준수 여부 확인
  - 1NF : 컬럼은 원자값인가
  - 2NF : 모든 속성이 PK 전체에 완전 종속되는가
  - 3NF : 비키 속성 간 이행적 종속 없는가
  - BCNF : 모든 결정자는 후보키인가
 
- 엔티티-관계 매핑 검증
  - 기본 체크리스트
    - 엔티티 : 핵심 업무 객체인가
    - 속성 : 의미 명확, 중복 없음?
    - 관계 : 1/1, 1/n, n/m 명확?
    - 식별자 : 고유 식별자 존재?
    - 무결성 : PK, FK, UNIQUE 정확?

#### 제약조건 검증
- 외래키 누락
- 잘못된 CASCADE 설정 : 의도치 않은 삭제
  - 상태값(is_deleted) 관리가 현실적 대안 / SOFT DELETE 할것
- NULL 제약조건 오설정

#### 성능 안티패턴
- 과도한 인덱스 생성
  - INSERT, UPDATE, DELETE 시 인덱스 유지 비용 증가
  - 조회 쿼리 성능 향상보다 관리 비용이 더 커질 수 있음
- 비효율적인 구조
  - LEFT JOIN 남발, 불필요한 조인
  - 조인 순서, 옵티마이저 계획 최적화 안됨
  - 데이터가 없는 상황에서도 JOIN으로 인해 Full Scan 발생
- 부적절한 데이터 타입 선택
  - 전화번호 INT (앞자리 0 사라짐) -> VARCHAR 사용 (앞자리 0 보존)
  - 소수값 없이 DECIMAL(10,0) -> INTEGER 사용
  - 짧은 상태값 CHAR(10) -> CHAR(1) 또는 ENUM 사용
  - 스토리지 낭비, 불필요한 캐스팅 필요
  - 검색 성능 저하 가능
  - 데이터 표현에 적합하지 않은 타입 사용 -> 향후 유지보수 시 문제
 
- 성능 최적화
  - 인덱싱
  - 역정규화(비정규화)
  - 쿼리튜닝 -> limit,offset으로 성능 최적화 가능
 
## 역정규화
- 정규화된 데이터베이스 구조에서 성능 또는 관리 편의성 등을 이유로 일부러 정규화를 완화하는 설계 기법
- 데이터 중복을 감수하고, 조인을 줄이거나 성능을 개선하기 위해 사용
- 복합 조회가 빈번하거나, 데이터 집계가 많은 대시보드, 통계 시스템에서 자주 활용
- 구분
  - 목적 : 성능 개선, 조인 최소화
  - 특징 : 일부 데이터 중복 허용
  - 주 용도 : 조회 성능 최적화, 복잡한 보고서 등
  - 데이터 일관성 : 수동 유지 필요 (트리거, 배치 등)
 
- 장점
  - JOIN 감소, 조회 속도 향상
  - 데이터 구조가 단순, 특정 쿼리에서 편리
  - 통계, 보고서 등 반복 조회 시 성능 유리
  - 읽기 성능 ↑ 쓰기 복잡도↓
- 단점
  - 갱신 복잡성 증가(UPDATE,DELETE 추가 작업 필요)
  - 데이터 중복으로 정합성 문제 발생 위험
  - 스키마 변경 시 영향 범위가 넓음
  - 읽기속도 <> 유지보수 비용 판단 필요
 
- 적용 기준
  - 테이블 조인 횟수가 많고, 성능 저하가 발생하는 경우
  - 자주 사용되는 복잡한 쿼리가 반복되는 경우 (대시보드, 보고서 등)
  - 데이터의 실시간성보다 빠른 조회가 중요한 경우 (캐시 개념)
  - 집계나 통계까 반복적으로 필요한 경우
 
- 적용 절차
  - 대상 테이블 및 성능 이슈 확인(실제 쿼리 비용 확인/시간+사용량)
  - 중복을 감수하고 추가할 컬럼 선정
  - 변경 이후 INSERT/UPDATE/DELETE 정책 설계
  - 인덱스 추가 등 튜닝 병행
  - 성능 테스트 및 롤백 계획 준비

- 성능 병목 구간을 명확히 파악한 후 적용
- 인덱스, 캐시, 뷰 등 다른 방법으로 해결 가능할지 먼저 고려

---

## ORM
- 객체 중심 기술

- SQL 중심 기술
  - 개발자가 SQL 쿼리를 직접 작성하여 데이터베이스에 접근하는 전통적인 방식
  - 별도의 XML Mapper 파일에 SQL 문장을 명시적으로 기술하며 개발자가 데이터베이스의 스키마와 구조를 완벽하게 이해하고 있어야 효율적인 개발이 가능
  - 특징
    - 직접 SQL 작성 : DBMS 최적화나 쿼리 튜닝을 위해 SQL을 직접 관리.
    - 명시적이고 직관적인 쿼리 : 복잡한 SQL을 그대로 사용 가능
    - SQL 의존성 : DBMS에 종속적인 SQL이 코드에 포함될 수 있음
    - 객체 매핑 수작업 필요 : 조회 결과를 JAVA 객체로 매핑하는 로직 필요
    - 변경 시 유지보수 어려움 : 스키마 변경 시 SQL을 전부 수정해야 할 수 있음
   
- 객체 중심 기술
  - 객체 단위로 데이터를 다룸 -> 객체와 테이블 간 자동 매핑
  - SQL을 직접 작성하지 않아도 대부분의 CRUD작업 가능
  - 영속성 컨텍스트(Persistence Context)를 통해 객체 상태 관리 (Dirty Checking, 1차 캐시 등)
  - 복잡한 비즈니스 로직에서 객체 상태 변화에 따라 DB 반영 -> 도메인 주도 설계(DDD)에 적합
  - 개발자가 비즈니스 로직에 집중 가능, DB 종속성 감소
  - 연관관계 매핑, Fetch 전략(Eager,Lazy) 등 다양한 기능 제공
 
## JPA
- Java 진영에서 사용하는 ORM 기술의 표준 명세 ( interface 추상화 )
- Java 객체와 관계형 데이터베이스의 데이터를 매핑하기 위한 표준 기술
- SQL 없이 자바 객체를 통해 테이터를 조작하는 방식
- 주요 특징
  - 표준 API : JDK에서 제공하는 공식 ORM 표준
  - 생산성 향상 : SQL을 직접 작성하지 않고 CRUD 가능
  - 유지보수 용이 : 객체 기반으로 코드 작성, SQL 분리로 코드 가독성 향상
  - DBMS 독립성 : SQL 의존 최소화, DB 변경 시 영향 최소화
  - ORM 기능 : 객체와 DB 테이블 자동 매핑
 
- Hibernate ORM
  - JPA를 구현한 대표적인 구현체
  - Application -> JPA Interface -> Hibernate-> JDBC -> DB

- JPA 동작 방식
  - 개념
    - Entity : DB 테이블과 매핑되는 JAVA 클래스
    - EntityManager : Entity의 저장, 조회, 삭제 등 관리 주체
    - Persistence Context : 1차 캐시, 엔티티 관리 공간(영속성 컨텍스트)
    - Transaction : 모든 데이터 변경은 트랜잭션 내에서 수행
   
  - 동작 단계
    - Entity 생성 -> persist()로 영속 상태 진입
    - 트랜잭션 커밋 시 실제 SQL 실행 -> DB반영
    - 조회 시 1차 캐시(Persistence Context) 우선 검색
   
- 영속성 컨텍스트
  - JPA가 엔티티 객체를 관리하는 메모리 공간 -> DB와 동기화되기 전 작업 중인 상태
  - 트랜잭션 커밋 시 DB에 저장 -> 컨텍스트 내부에 데이터를 여전히 남겨둠
  - 조회 시 컨텍스트에 있으면 DB에 액세스 하지 않고 가져옴 (1차 캐시)
 
#### JPA 구성요소
- Entity
  - DB 테이블과 매핑되는 자바 클래스
  - @Entity 어노테이션으로 명시
  - @Id : Primary Key 지정
  - @GeneratedValue: PK 자동 생성 전략
 
- EntityManager
  - JPA 핵심 인터페이스
  - **Entity의 생명주기 관리, CRUD, JPQL 실행 등을 담당**
 
- 영속성 컨텍스트
  - 테이블과 매핑되는 엔티티 객체 정보를 보관하는 장소
  - 보관된 엔티티 정보는 데이터베이스 테이블에 데이터를 저장, 수정, 조회, 삭제하는데 사용
  - 1차 캐시 영역과 쓰기 지연 SQL 저장소 영역 존재
  - EntityManager.persist()를 통해 Entity를 1차 캐시 영역에 저장
  - EntityTransaction.commit()을 해야 쓰기 지연 SQL 저장소에 저장된 SQL들이 실행됨 (LAZY)
 
- JPQL
  - Java Persistence Query Language
  - SQL과 유사하지만, Entity를 대상으로 질의
  - DB가 아닌 객체 중심 언어
  - ```java
    // 모든 회원 조회 예시
    List<Member> members = em.createQuery("SELECT m FROM Member m", Member.class)
                              .getResultList();
    ```

## Spring Data JPA
- Spring Framework에서 제공하는 데이터 접근 기술
- 반복적인 CRUD 코드의 생략과 표준화된 인터페이스를 통해 생산성과 유지보수성을 극대화
- 역할
  - Repository 패턴을 통한 데이터 접근 표준화
  - JPA 위에 얹혀 동작(JPA 필수 선행)
 
- 동작 구조
  - Repository Interface 작성 (인터페이스만 작성, 구현 필요 없음)
  - Spring Data JPA가 내부적으로 Proxy 객체 생성
  - EntityManager 사용하여 JPA 동작 위임
 
- 주요 기능
  - JpaRepository 상속 : 기본 CRUD 메서드 제공
  - Query Method : 메서드 이름으로 쿼리 자동 생성
  - @Query : JPQL 작성 가능
  - Native Query : 실제 DB SQL 사용 가능
  - 페이징/정렬 : Pageable, Sort 인터페이스 제공
  - Projection : 일부 컬럼만 조회 가능
 
- 복잡한 조건은 @Query로 (JPQL)
  - @Query(value=[sql], nativeQuery=true) / SQL로 @Query 어노테이션 사용하기

- 자동구현
  - Repository 인터페이스 작성 (JpaRepository 상속)
  - Spring Boot 실행 시 프록시 객체 생성
  - 메서드 이름 분석 -> JPQL 생성
  - EntityManager 통해 DB 조회
  - https://docs.spring.io/spring-data/jpa/reference/#repositories.query-methods.details / 자동 쿼리 생성 규칙
 
#### Entity
- @Entity 어노테이션 추가
  - @NoArgsConstructor 필수
  - @Id 필수 (기본키)
  - @Table은 옵션
  - 객체의 이름과 테이블 명을 다르게 할 수 있음 @Entity(name = ""), @Table(name = "")

- 기본키 매핑
  - 기본키를 어떤 방식으로 생성해 줄지에 대한 다양한 전략 지원
  - 기본키 직접 할당
    - 애플리케이션 코드 상에서 기본키를 직접 할당해주는 방식
  - 기본키 자동 생성
    - IDENTITY
      - 기본키 생성을 데이터베이스에 위임하는 전략
    - SEQUENCE
      - 데이터베이스에서 제공하는 시퀀스를 사용해서 기본키를 생성하는 전략
    - TABLE
      - 별도의 키 생성 테이블을 사용하는 전략
     
- @Column
  - 애트리뷰트
    - nullable
    - updatable
    - unique
    - Date,Calendar 타입으로 매핑하기 위해서는 @Temporal 필요 / LocalDateTime은 필요없음 -> TIMESTAMP 타입과 매핑
    - name
   
- @Enumerated
  - enum 타입과 매핑할 때 사용하는 어노테이션
  - EnumType.ORDINAL / enum의 순서를 나타내는 숫자를 테이블에 저장
    - enum의 순서가 뒤바뀔 가능성도 있으므로 처음부터 EnumType.STRING 사용 권장
  - EnumType.STRING / enum의 이름을 테이블에 저장

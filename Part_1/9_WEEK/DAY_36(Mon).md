# DAY_36
---
## 목차
- [이론](#이론)
- [데이터베이스](#데이터베이스)
- [SQL](#sql)
  - [DDL](#ddl)
  - [DML](#dml)
  - [관계형 데이터베이스](#관계형-db)
  - [postgreSQL](#postgresql)
  - [SQL 구문 기본 구조](#sql-구문-기본-구조)

---

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
  - 특히 Dto 필드(파라미터) 명이 다를 경우 바인딩 되지 않는 것을 찾는 것이 까다로웠다.
  - 기존에는 ModelAttribute로 파일과 Dto를 동시에 입력받고 있었는데 이것을 RequestPart로 수정하는 부분에서 MultipartFile이 입력되지 않는 상황이 굉장히 많은 시간을 소요한 것 같다.
    - 매핑은 Form-Data로 매핑을 받고 Dto는 어노테이션을 사용하여 새롭게 정의해서 JSON으로 입력 받도록 하는 과정이 많은 시간을 소요시켰다.
    - 커스텀Exception을 추가하였고 ExceptionCode를 추가하여 로깅을 좀 더 쉽게 하도록 수정하였다.

---

# 이론

## 데이터베이스
- 구조화된 데이터를 효율적으로 저장, 관리, 검색 할 수 있는 시스템
- 파일 시스템 기반 저장 방식의 한계
  - 데이터 중복 / 여러 파일에 동일한 데이터가 저장되면서 관리가 어려움
  - 데이터 무결성 / 서로 다른 파일에서 동일한 데이터가 다르게 기록될 수 있음
  - 검색 성능 저하 / 파일을 일일이 열어 내용을 분석해야 함
  - 동시성 처리 불가 / 여러 사용자가 동시에 접근하면 데이터 손상 위험
  - 백업 및 복구 어려움 / 수동으로 처리해야 하므로 복구 시간 지연 발생

-  데이터베이스 관리 시스템 (DBMS)의 등장
  - 데이터를 체계적으로 관리하고, 효율적이고 안전하게 처리할 수 있게 도와주는 소프트웨어
  - 주요 기능
    - 데이터 저장, 검색, 수정, 삭제 지원
    - 트랜잭션 처리
    - 동시성 제어
    - 데이터 무결성 및 보안 강화
    - 백업 및 복구 기능 내장
   
- RDB
  - 데이터를 테이블 형태로 저장하며, 각 테이블은 열과 행으로 구성

| id | name       | email            |
|----|------------|------------------|
| 1  | John Doe   | john@example.com |
| 2  | Jane Smith | jane@example.com |

  - 데이터를 행 단위로 저장
  - 열은 데이터의 속성(Attribute) 정의
  - 테이블 간의 관계는 키를 통해 연결됨

- 실무에서 DB를 사용해야 하는 이유
  - 유지보수 효율
  - 확장성과 성능
    - 대량 데이터도 빠르게 검색(인덱스, 최적화 쿼리)
  - 협업 가능성
    - 팀원 간 공통된 규약으로 데이터 공유 가능
    - 데이터를 API와 연동하여 여러 시스템과 통합 가능
   
## SQL
- Structured Query Language
- RDB에서 데이터를 정의하고, 조작하며, 제어할 수 있도록 설계된 선언형 언어
- ANSI(미국표준협회)와 ISO(국제표준화기구)에 의해 국제 표준으로 제정
- 목적과 역할
  - 데이터를 정형화된 구조로 관리
  - 사용자가 무엇을 하고 싶은지를 선언적으로 표현 (절차는 DBMS가 처리)
  - 프로그래밍 언어와 결합되어 다양한 애플리케이션에서 데이터 접근 가능
 
- 특징
  - 표준화된 언어 : ANSI/ISO 표준에 따라 일관된 문법과 기능 제공
  - 다양한 하위 명령어 그룹
    - DDL(Data Definition Language) : 테이블, 뷰, 인덱스 등의 구조 정의 / CREATE, ALTER, DROP
    - DML(Data Manipulation Language) : 테이블의 데이터 삽입, 조회, 수정, 삭제 / SELECT, INSERT, UPDATE, DELETE
    - DCL(Data Control Language) : 사용자 접근 권한 관리 / GRANT, REVOKE
    - TCL(Transaction Control Language) : 트랜잭션 처리 및 롤백 / COMMIT, ROLLBACK 등
  - 범용성 : 거의 모든 RDMBS에서 사용되며, 백엔드 API, 통계 시스템, BI 도구 등과 연결 가능
  - 확장성 : 표준 SQL 위에 각 DMBS가 필요에 따라 확장문법 을 추가 구현
 
- 다양한 DMBS를 고려한 SQL 작성할 때는 공통된 표준 문법만 사용하는 것이 중요

#### DDL
- 스키자(구조)를 정의하거나 변경할 때 사용하는 명령어
- 주로 테이블 생성, 삭제, 컬럼 변경 등에 사용
- 일반적으로 자동 커밋이 수행
- 생성
- ```sql
  CREATE TABLE member (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```
- 컬럼 추가
- ```sql
  ALTER TABLE member ADD phone_number VARCHAR(20);
  ```
- 컬럼 수정
- ```sql
  ALTER TABLE member MODIFY name VARCHAR(200);
  ```
- 컬럼 삭제
- ```sql
  ALTER TABLE memeber DROP COLUMN phone_number;
  ```
- 테이블 이름 변경
- ```sql
  ALTER TABLE member RENAME TO user;
  ```
- 테이블 삭제
- ```sql
  DROP TABLE user;
  ```
- 테이블 비우기
- ```sql
  TRUNCATE TABLE user;
  ```
- 대부분의 DBMS에서 ALTER는 성능 부하가 있음. 운영 환경에서는 최소한으로 사용해야 함
- TRUNCATE는 트랜잭션 로그를 거의 남기지 않아 롤백이 불가 ( DROP도 )

#### DML
- 테이블에 저장된 데이터 자체를 조회하거나 변경 하는데 사용하는 명령어
- 전체 조회
- ```sql
  SELECT * FROM member;
  ```
- 조건부 조회
- ```sql
  SELECT id, name FROM member WHERE email LIKE '%@example.com';
  -- @example.com 도메민을 사용하는 회원만 조회
  ```
- 정렬하여 조회
- ```sql
  SELECT * FROM member ORDER BY created_at DESC;
  ```
- 데이터 삽입
- ```sql
  INSERT INTO member (id, name, email) VALUES (101, '홍길동', 'hong@example.com');
  ```
- 다건 삽입
- ```sql
  INSERT INTO member (id, name, email) VALUES (102, '이몽룡', 'lee@example.com'), (103, '성춘향', 'sung@example.com');
  ```
- 특정 사용자 수정
- ```sql
  UPDATE member SET name = '홍길자' WHERE id = 101;
  ```
- 전체 수정
- ```sql
  UPDATE member SET name = '사용자';
  ```
- 데이터 삭제
- ```sql
  DELETE FROM member WHERE id = 101;
  ```
- 테이블 내 모든 데이터 삭제
- ```sql
  DELETE FROM member;
  ```
- INSERT 시 컬럼 순서와 값 순서를 정확히 맞춰야 함
- SELECT 문은 실제 운영 환경에서는 인덱스와 결합하여 최적화 필요
- DELETE 또는 UPDATE에서 WHERE 조건을 빼먹는 경우 대규모 데이터 손실로 이어질 수 있다. 반드시 조건 확인 절차 마련

#### 관계형 DB
- 데이터를 테이블 형식으로 저장하고 이 테이블들 간에 관계를 설정하여 데이터를 효율적으로 관리하는 구조
- 구성 요소
  - 테이블 : 하나의 주제를 표현하는 데이터 집합
  - 행 : 데이터 한 건
  - 열 : 속성 또는 데이터의 항목
  - 키 : 데이터 간 관계를 연결하기 위한 식별자(ex. 기본키, 외래키)
 
- 핵심 특징
  - 정형화된 구조
    - 모든 데이터는 명확한 행과 열 형태로 저장된다.
    - 데이터 스키마(구조)가 명확하게 정의되어 있어 데이터 무결성 유지에 용이
  - 무결성
    - 엔티티 무결성 : 기본키는 중복되거나 null일 수 없다
    - 참조 무결성 : 외래키는 반드시 참조 대상의 기본키를 가져야 합니다.
    - ```sql
      CREATE TABLE Orders (
        id INT PRIMARY KEY,
        user_id INT,
        FOREIGN KEY (user_id) REFERENCES Users(id)
      );
      ```
  - 관계 기반설계
    - 테이블 간의 관계를 기본키/외래키를 통해 명시적으로 정의
    - 1:1,1:n,n:m 관계를 설정할 수 있어 중복 최소화 및 데이터 분리에 유리
  - 정규화
    - 데이터를 중복 없이 분해하고, 재사용 가능하도록 설계하는 과정
    - 1NF, 2NF, 3NF 등의 정규화 단계가 존재하며, 이를 통해 데이터 일관성과 관리 효율성을 높일 수 있다.
      - 1NF : 컬럼은 원자값만 저장 ( 더이상 쪼개지지 않는 정보 , ex. 서울특별시 강남구 -> 서울특별시 컬럼과 강남구 컬럼으로 나눔)
      - 2NF : 부분 종속 제거 (복합 키 -> 단일 키)
      - 3NF : 이행 종속 제거 (비기본 속성이 다른 비기본 속성에 의존 X)
  - SQL을 통한 표준화된 데이터 처리
- 실무에서 RDB를 사용하는 이유
  - 데이터 정확성 복장
  - 관계 기반 연산
  - 스키마 기반 관리
  - 안정적인 트랜잭션 처리
 
#### PostgreSQL
- 객체-관계형 데이터베이스 관리 시스템(ORDBMS), 오픈소스이면서 신뢰성과 확장성을 갖춘 대표적인 RDBMS
- 트랜잭션을 지원하고 ACID 특성 보장
- JSON, 배열, GIS 등 다양한 데이터 타입 지원
- 커스터마이징이 용이하고 대규모 서비스에도 사용 가능
- MySQL 대비 더 엄격한 SQL 준수

---

- DB접근 도구
  - 기능 지원
    - 테이블 구조와 관계 시각화
    - 실시간 쿼리 실행 및 결과 확인
    - DB 상태 진단
    - 사용자 계정, 권한, 스키마 관리
    - 다중 환경 통합 관리
   
#### SQL 구문 기본 구조
- 키워드 : SQL의 명령과 흐름 제어를 위한 예약어
  - 미리 정의됨. 일반적으로 대문자로 작성하는 것이 관례
  - DML, DDL, 제어문, 조건문
- 식별자 : 테이블명, 컬럼명 등 데이터베이스 객체 이름
  - 테이블, 컬럼, 인덱스, 제약조건 등 객체의 이름을 의미
  - 대소문자 구분 여부 / 대부분 구분하지 않음 / 쌍따옴표로 감싸면 구분
  - 예약어 사용 제한
  - 허용 문자 / 알파벳, 숫자, 밑줄 가능. 숫자로 시작하거나 특수문자 포함 시 쌍따옴표 필요, 공백 불가
- 연산자 : 조건 비교 또는 산술 계산
- 리터널 : 고정된 값
- 주석 : 문장 설명용 비실행 구문 / -- 한줄 주석 //* 여러줄 주석 *//

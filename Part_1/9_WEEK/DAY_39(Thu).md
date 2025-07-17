# DAY 39
---
## 목차
- [Entity 연관관계](#entity-연관관계)
- [JpaRepository](#jparepository)
  - [@Query](#@query)
- [Page](#page)
  - [Page, Slice](#page-slice)
- [Sort](#sort)
- [N+1 문제](#n+1-문제)
  - [Fetch Join](#fetch-join)
  - [EntityGraph](#entitygraph)
- [즉시로딩과 지연로딩](#즉시로딩과-지연로딩-(eager,-lazy))
- [트랜잭션](#트랜잭션)
- [ACID](#acid)
- [트랜잭션 상태와 생명주기](#트랜잭션-상태와-생명주기)

---

## Entity 연관관계 매핑
- 단방향 연관관계
  - Member 클래스가 List<Order>를 가지고 있던가
  - Order 클래스가 Member 객체를 가지고 있는 관계
  - 한쪽은 다른 한쪽의 정보를 가지고 있지만 다른 한쪽은 정보를 가지고 있지 않는 관계
 
- 양방향 연관 관계
  - Member 클래스가 List<Order>를 가지고 있으며, Order 클래스가 Member 객체를 가지고 있는 관계
  - 서로가 서로의 객체를 참조할 수 있는 관계
 
- 일대다 단방향 연관 관계
  - 위에서 설명한 Member-Order관계가 1:n(일대다) 관계
  - DFD에서 일대다 연관관계를 정상적으로 표현하지 못해서 잘 사용되지 않는다.
  - 보통 DB는 n이 1의 PK를 가지고 있기 때문에 Member가 Order의 정보를 가지고 있지 않기 때문
  - 다대일 단방향 매핑을 먼저 한 후 필요에 의해서 일대다 단방향 매핑을 추가해 양방향 연관 관계를 만들어 주는 것이 일반적

- 다대일 단방향 연관 관계
  - 테이블 간의 관계처럼 자연스러운 매핑 방식
  - JPA에서 가장 기본으로 사용되는 매핑 방식
  - 다(n)에 해당하는 클래스가 일(1)에 해당하는 객체를 참조할 수 있는 관계
 
- 연관관계 코드
  - @ManyToOne 다대일 관계 명시
  - @JoinColumn(name = '') 외래키에 해당하는 열의 이름
  - @OneToMany(mappedBy = '') 참조중인 클래스의 ManyToOne의 필드명 / 이 관계의 외래키는 나한테 없고 다른쪽 필드가 관리한다.
  - 순환참조 끊기
    - @JsonManagedReference
    - DTO 사용 / Mapper에서 끊기
   
- 연관관계의 주인
  - 외래키가 어디 있는지
  - 양방향 연관관계에서 외래키를 관리하는 쪽을 연관관계의 주인이라 부른다.
  - 주인이 아닌 쪽 -> mappedBy 사용
 
- 영속성 전이
  - 연관관계 편의 메서드
  - ```java
    public void addMember(Member member) {
      this.member = member;
      if (member.getStamp == null) {
        member.addStamp(this);
      }
    }
    //
    public void addStamp(Stamp stamp) {
      this.stamp = stamp;
      if(stamp.getMember==null) {
        stamp.addMember(this);
      }
    }
    ```
  - OneToOne(cascade = CASCADETYPE.[])
 
## JpaRepository
- 데이터 접근 계층을 매우 단순하게 만들어주는 인터페이스
- CrudRepository -> PagingAndSortingRepository -> JpaRepository로 이어지는 상속구조를 가진다.
- public interface MemberRepositoy **extends JpaRepository<Member,Long>** {}
- 내부적으로 SimpleJpaRepository 구현 사용.
- SimpleJpaRepository가 EntityManager를 사용하여 DB 접근 로직 수행
- 프록시 패턴으로 런타임에 Spring이 구현체를 만들어 주입
  - Spring AOP 기반 동적 프록시를 생성.
  - JpaRepositoryFactory가 만든 프록시 객체가 실제 개발자에게 주입됨.
 
- 기본 제공 메서드
  - save()
  - findById()
  - count()
  - delete()
  - existsById()
  - findAll(Pageable), findAll(Sort)
  - ...
 
- 쿼리 메서드 작성
  - findBy...
  - countBy...
  - deleteBy...
  - existsBy...
  - TopN... 상위 N개
    - findTop3ByOrderBy...
  - First... 첫 번째
    - findFirstBy...
  - And, Or 가능
  - GreaterThan, LessThen, GreaterThanEqual... 가능

- 복잡한 쿼리는 QueryDSL이나 @Query로 진행
- 단순 조건에 사용
- Optional 반환으로 NPE 방지

- 복잡한 조건은 자동 생성 메서드를 사용하면 성능이 매우 떨어질 수 있음.
- 복잡한 조건은 QueryDSL 이나 @Query 어노테이션 사용

#### @Query
- JPQL이나 Native Query 작성
- JPQL
  - ```java
    @Query("SELECT m FROM Member m WHERE m.username = :username")
    Member findByname(@Param("username") String username);
    ```
  - Entity 기반
- Native Query를 사용할때는
  - @Query() 어노테이션 어트리뷰트로 **value**에 SQL, **nativeQuery**=true로 설정해야함.
  - ```java
    @Query(nativeQuery=true, value="SELECT * FROM member WHERE name = :username")
    Member findByname(@Param("username") String username);
    ```
  - DB 테이블 기준
 
## Pageable
- 페이징을 위한 정보를 담는 인터페이스
- 페이지 번호, 페이지 크기, 정렬 정보를 포함
- PageRequest.of()로 간편하게 인스턴스 생성
- 0부터 시작하는 페이지 인덱스 사용
- 주요 메서드
  - getPageNumber() : 현재 페이지 번호
  - getPageSize() : 한 페이지 당 데이터 수
  - getSort() : 정렬 정보 반환
  - next() : 다음 페이지 Pageable 반환
  - previousOrFirst() : 이전 페이지 또는 첫 페이지 Pageable 반환
 
- Pageable pageable = PageRequest.of(0, 10, Sort.by("username").descending());
- 0 페이지, 한 페이지당 10개, username으로 정렬.내림차순

#### Page Slice
- Page
  - 전체 데이터 개수/ 총 페이지 필요할 때 사용
    - 관리자용 대시보드, 관리화면
      - 총 페이지 수를 보여줘야 함.
      - 사용자가 1/10 페이지 이런 UI를 보고 사용
    - 검색 기능
      - 총 123건 중 5페이지 와 같은 정보를 보여줘야 함
  - 1페이지 -> 7페이지 -> 3페이지 페이지를 선택해서 보여줄 때 필요
- Slice
  - 다음 페이지 여부만 확인하면 될 때 사용
    - 모바일 앱 - 무한 스크롤
      - 다음 페이지 있으면 스크롤 더 내려서 자동 로드
      - 전체 데이터 개수 필요 없음, 성능 최우선
    - 더보기 버튼
      - "더보기" 눌렀을 때 다음 페이지 있으면 가져옴
      - 마지막 페이지인지 여부만 중요 
  - count 쿼리를 사용하지 않아 빠름
  - 내부적으로 limit + 1로 조회
    - 요청보다 1개 더 조회 -> 다음 페이지 존재 여부만 판단

## Sort
- 정렬 조건을 명시적으로 설정하는 인터페이스
- 여러 필드에 대한 정렬 지정 가능
- Sort.by()를 통해 여러 필드 정렬 가능
- 오름차순, 내림차순 지정 가능

## N+1 문제
- ORM 환경에서 1번의 쿼리로 N개의 데이터를 조회한 후, 각 데이터의 연관된 데이터를 추가 조회하면서 총 N+1번의 쿼리가 발생하는 문제
- 지연 로딩 전략으로 연관 관계가 설정된 엔티티를 조회할 때, 각각 개별적으로 추가 조회가 발생하기 때문
- 문제점
  - 성능 저하
  - 복잡도 증가
  - 예측 어려움
- 컬렉션의 Lazy 로딩은 특히 조심
- Fetch Join, EntityGraph로 해결

#### Fetch Join
- JPQL에서 JOIN FETCH 구문을 사용하면, 연관된 엔티티를 한 번에 조회
- ```
  SELECT m FROM Member m JOIN FETCH m.orders
  ```
- **페이징 불가(데이터 부풀려짐)**
- OneToMany Fetch Join 시 **중복 발생** -> DISTINCT / 컬렉션에서 사용 시 DISTINCT 권장
 
#### EntityGraph
- 쿼리를 작성하지 않고 fetch 전략을 재정의할 수 있는 방법
- 특정 쿼리에만 Eager처럼 작동하게 변경 가능
- JPQL 없이 선언적으로 연관 엔티티 조회 가능
- @EntityGraph(attributePaths = {"orders"})
- 페이징 시에도 사용 가능

## 즉시로딩과 지연로딩 (EAGER, LAZY)
- 즉시로딩
  - 연관된 엔티티를 즉시 함께 조회
  - Eager는 즉시 가져오라는 의미 -> JOIN하라가 아님 -> N+1 발생 가능
  - 컬렉션 관계의 특성상 JOIN 불가 -> EAGER라도 연관된 컬렉션은 N번 조회할 수 밖에 없음 (OneToMany)

- 지연로딩
  - 실제로 사용할 때 쿼리가 실행 -> 초기 조회 시에는 프록시 객체만 주입
 
## 영속성 전이
- 부모 엔티티의 영속 상태 변경 이 자식 엔티티에 전이되도록 도와주는 기능
- 부모를 저장하거나 삭제하면 자식까지 함께 저장/삭제되도록 한다.
- 트랜잭션 단위의 일관성 유지 : 부모와 자식은 하나의 단위로 관리되므로 관리 실수를 방지할 수 있다.

- Cascade 종류
  - PERSIST : 자동 저장
  - MERGE : 자동 병합
  - REMOVE : 자동 삭제
  - REFRESH : 자동 초기화
  - DETACH : 영속성 컨텍스트에서 분리 (더 이상 JPA가 추적하지 않도록)
  - ALL
 
- 흔히 PERSIST와 REMOVE 사용
- OneToMany(mappedBy = "parent", cascde = {CascadeType.PERSIST, CascadeType.REMOVE})

- 고아 객체 제거
  - orphanRemoval
    - **컬렉션**에서 엔티티가 제거되면 자동으로 DB에서 삭제해주는 기능
    - 부모가 자식을 더 이상 참조하지 않는다면, DB에 남지 않도록 자동 삭제
    - 영속성 컨텍스트안에 존재할 때 사용가능 ( 영속성 관계 안에서만 적용 )
  - 부모에 속한 자식이 따로 존재 의미가 없을 때 사용
   
## 트랜잭션
- 데이터베이스 상태를 변화시키는 작업의 논리적 단위
- 모두 성공하거나 모두 실패해야 함, 도중에 일부만 실행되는 일이 없어야 한다.
  - **데이터 무결성(일관성) 보장**
- 속성 (ACID)
  - 원자성 : 모두 성공 또는 모두 실패
  - 일관성 : 트랜잭션 전후 데이터 무결성 유지
  - 고립성 : 동시에 실행되는 트랜잭션 간 영향 없음
  - 지속성 : 성공된 데이터는 영구 반영
 
- 데이터 무결성 보장과 예외 상황 대비
  - 복구 가능성 항상 고려해야 함
  - 모든 DB는 ACID 보장을 위한 기능 제공
 
- 트랜잭션 흐름
  - 정상 흐름
    - 트랜잭션 시작
    - 모든 작업 성공
    - Commit -> 데이터 영구 반영
   
  - 실패 흐름
    - 트랜잭션 시작
    - 중간에 예외 발생
    - Rollback -> 이전 상태로 복원
   
- 미사용 시 개별 작업은 성공했으나, 전체 흐름은 실패하는 상황 발생

## ACID
- A Actomicity(원자성)
  - 트랜잭션 내 모든 작업은 하나의 단위로서 완전하게 수행되거나, 전부 수행되지 않아야 한다.
 
- C Consistency(일관성)
  - 논리적 모순이 없음, 자체적 일치, 예측 가능한 상태 유지
  - 트랜잭션 전과 후에 데이터 무결성 제약조건을 반드시 만족해야 한다.
 
- I Isolation(격리성)
  - 각 트랜잭션은 서로 영향을 받지 않도록 분리되어야 한다.
  - ex: 동시에 같은 상품 구매 시, 재고 차감이 꼬이지 않도록 트랜잭션 격리 필요
  - 격리 수준
    - READ UNCOMMITTED : 다른 트랜잭션의 변경 내용을 읽음 (Dirty Read 허용)
    - READ COMMITTED : 커밋된 데이터만 읽음
    - REPEATABLE READ : 같은 데이터를 항상 동일하게 읽음
    - SERIALIZABLE : 가장 엄격, 트랜잭션 순차 실행
   
  - Dirty Read : 다른 트랜잭션이 아직 COMMIT하지 않은 데이터를 읽는 것
 
- D Durability(지속성)
  - 트랜잭션이 성공적으로 완료되면, 그 결과는 DB에 영구적으로 반영되어야 함.
  - DB의 보장 방법
    - WAL(Write Ahead Log) : PostgreSQL은 변경사항을 먼저 로그에 기록후 반영
    - 장애 발생 시 WAL을 통해 복구
   
## 트랜잭션 상태와 생명주기
- 상태
  - BEGIN : 시작 @Transactional, BEGIN
  - ACTIVE : 작업 수행
  - PARTIALLY COMMITTED : 커밋 직전
    - COMMITTED : 성공
    - FAILED / ABORTED : 실패 / FAILED = 오류 발생 상태 , ABORTED = 변경사항 취소된 상태
  - END
 
## 선언적 트랜잭션
- @Transactional 어노테이션을 선언하는 방식
- 클래스 레벨과 메서드 레벨에서 적용 가능
- 클래스 레벨에 사용 시 모든 **public** 메서드에 적용 / 프록시 객체는 private에 접근할 수 없음
- 메서드 레벨에 적용 시 메서드에 설정이 적용 됨

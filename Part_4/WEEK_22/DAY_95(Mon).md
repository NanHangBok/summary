# DAY_95

## 목차
- [CompletableFuture](#completablefuture)
- [Spring 비동기](#spring-비동기)
- [TaskExecutor](#taskexecutor)
- [Spring Event](#sprint-event)
- [비동기 예외 처리](#비동기-예외-처리)
- [TaskDecorator](#taskdecorator)
- [WebClient](#webclient)
---

## CompletableFuture
#### 비동기 작업 조합의 필요성
  - 단일 작업만 처리하는 것이 아니라, 여러 비동기 작업을 연결하거나 결합할 수 있음
  - 이 때 사용하는 핵심 메서드
    - thenCompose() 순차 실행 / 결과 -> 다음 작업 입력
    - thenCombine() 병렬 실행 후 결합 / 두 Future 결과 결합
    - allOf() 다수 작업 모두 완료 시 / 전체 완료 대기
    - anyOf() 다수 작업 중 하나 완료 시 / 첫 완료 즉시 반환

- thenCompose() - 순차적 비동기 연결
  - 이전 비동기 결과를 받아 다음 비동기 작업으로 연결할 때 사용
  - 비동기 작업을 순차적으로 연결할 때 사용
  - 내부적으로 Future의 중첩 구조를 평탄화 함
    - thenApply로 연결하면 Future<Future<T>> 구조가 됨.
    - ```java
      CompletableFuture<CompletableFuture<String>> wrong = CompletableFuture.supplyAsync(() -> "A")
        .thenApply(a -> CompletableFuture.supplyAsync(() -> a + "B"));
      ```
- thenCombine() - 병렬 비동기 결합
  - 두 개의 독립적인 비동기 작업을 병렬로 실행하고
  - 두 결과를 결합하여 새로운 결과 생성
  - 서로 독립적인 두 Future를 병렬로 수행 한 뒤, 두 결과를 하나의 결과로 합치는데 사용
 
- allOf() - 여러 작업 모두 완료 대기
  - 여러 비동기 작업이 모두 완료될 때 까지 기다림
  - 반환값이 없으며, 모든 작업이 끝난 시점을 감지할 때 사용
  - 반환값이 필요한 경우
  - ```java
    List<CompletableFuture<Integer>> futures = List.of(
        CompletableFuture.supplyAsync(() -> 10),
        CompletableFuture.supplyAsync(() -> 20),
        CompletableFuture.supplyAsync(() -> 30)
    );
    
    CompletableFuture<Void> all = CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]));
    
    // 모든 Future 완료 후 결과 수집
    List<Integer> results = futures.stream().map(CompletableFuture::join).toList();
    System.out.println(results); // [10, 20, 30]
    ```
- anyOf() - 여러 작업 중 하나만 완료되면 반환
  - 여러 비동기 작업 중 가장 먼저 완료된 결과를 반환
  - 첫 번째로 끝난 결과만 필요할 때 유용
    - ex: 여러 서버 중 가장 빠른 응답을 사용하는 경우

#### 예외 처리
- 비동기 프로그램에서는 스레드 내부에서 예외가 발생해도 실시간에 외부로 전달되지 않음
  + 이 때문에 단순한 try-catch문으로는 예외를 포착할 수 없음
  + 대신 CompletionException 이라는 새로운 예외로 감싸져(Wrapping) 던져짐
- ComplatableFuture는 내부 스레드에서 발생한 모든 예외를 CompletionException으로 래핑하여 메인 스레드로 전달 됨.
  + join() 또는 get()을 호출할 때 예외가 CompletionException으로 감싸져서 발생
  + 내부 스레드에서 발생한 예외는 즉시 메인 스레드로 전파되지 않음
  + try-catch로 직접 잡을 수 없기 때문에, exceptionally(), handle(), whenComplete() 같은 메서드를 사용해야 함
 
- exceptionally - 예외 발생 시 대체값 반환
  + 예외가 발생했을 때만 호출되는 복구 컬백
  + 정상적으로 완료되면 무시되고, 오류 건은 따로 처리
 
- handle() - 정상/예외 모두 처리
  + 성공과 실패를 모두 처리할 수 있는 콜백
  + 매개변수 2개 :
    + 첫 번째, 정상 결과값 (예외 시 null)
    + 두 번째, 예외 객체 (정상 시 null)
   
- whenComplete() - 결과 유지 + 후 처리
  + 결과를 변경하지 않고, 후처리(로깅, 리소스 정리 등)에 사용 됨.
  + 결과는 그대로 다음 단계로 전달 됨
 
#### 타임아웃
- 비동기 프로그램에서는 외부 서비스 지연, 네트워크 문제, 무한 대기 상태와 같은 상황이 빈번하게 발생함
  + 이때 타임아웃 설정을 하지 않으면, 프로그램은 결과를 무한히 기다리게 되어 시스템 전체의 응답성이 저하 됨
  + 핵심 목적
    + 외부 의존성(API, DB 등)의 지연으로부터 시스템 보호
    + 사용자의 응답 대기 시간 제한
    + 장애 허용 전략 구현
    + 복구 로직 트리거 역할 수행
   
- orTimeout() - 일정 시간 내 완료되지 않으면 예외 발생
  + 지정한 시간이 지나도 Future가 완료되지 않으면 예외를 발생
  + TimeoutException 발생. 내부적으로 completeExceptionally()가 호출 됨
  + 결과를 대체하지 않고 예외를 던짐
    + exceptionally()나 handle()과 조합하여 예외를 복구할 수 있음

- completeOnTimeout() - 타임아웃 시 기본값 반환
  + 지정한 시간이 지나도 완료되지 않으면 예외 대신 기본값을 반환
  + 안전한 fallBack 값을 제공하는 역할을 함
  + 타임아웃 발생 시 예외를 던지지 않음
  + 기본값으로 Future를 정상 완료 시킴
  + 안정성이 요구되는 서비스 (Fallback 처리)에 적합
 
| 구분 | orTimeout() | completeOnTimeout() |
|-|-|-|
| 타임아웃 발생 시 | 예외 발생(TimeoutException) | 기본값으로 완료 |
| 프로그램 흐름 | 실패로 간주 | 정상 완료로 간주 |
| 반환 타입 | 기존 타입 유지 | 기존 타입 유지 |
| 예외 복구 필요 여부 | 필요함 | 불필요함 |
| 주 사용 목적 | 오류 감지 및 예외 처리 | Fallback 결과 제공 |


#### 대용량 처리 비동기 배치
- 대량의 데이터를 처리할 때, 단일 스레드로 순차 처리하면 성능이 급격히 저하됨.
- 이때 CompletableFuture를 이용하면 데이터를 여러 덩어리(Batch)로 나누어 동시에 처리할 수 있음
- 너무 많은 스레드가 한 번에 생성되면 문맥 전환 비용이 발생하고, 시스템 성능이 떨어짐.
  + 따라서 아래처럼 고정 스레드 풀을 사용하여 병렬 실행 개수를 제한하는 것이 중요

- 병렬 스트림
  + paralleStream() 으로도 병렬 처리를 쉽게 구현 가능
  + 하지만 CompletableFuture가 병렬 수준 제어, 예외 처리, 체이닝 등 훨씬 강력한 기능 제공.

| 비교 항목 | ParallelStream | CompletableFuture |
|-|-|-|
| 스레드 제어 | 불가능(ForkJoinPool 공용 사용) | 가능 (Executor 직접 지정) |
| 예외 처리 | 불편(try-catch 필요) | exceptionally, handle 가능 |
| 비동기 체인 | 불가능 | 체인형 비동기 조합 지원 |
| 제어력 | 낮음 | 높음 |

## Spring 비동기
- 대용량 트래픽 환경에서의 성능 개선
  + 스레드의 요청이 오래 걸리면 그만큼 대기(스레드의 낭비)가 발생 -> 성능 저하
    + 요청을 백그라운드 스레드에 위임하고, 메인스레드는 즉시 다음 요청을 처리(비동기 처리)
- 사용자 경험 향상 (응답 시간 최적화)
  + 사용자는 빠른 응답을 가장 먼저 체감
    + 3초 기다리는 대신, 즉시 "요청이 접수되었습니다" 라는 메시지를 받는다면 만족도가 높아짐
   
- Spring 비동기 처리
  + Spring이 ThreadPoolTaskExecutor로 관리
  + 메서드에 @Async 한 줄만 추가
  + AsyncUncaughtExceptionHandler 자동 관리
  + 트랜잭션, 보안, AOP 통합 가능
 
- 핵심 구성 요소

| 구성 요소 | 역할 |
|-|-|
| @Async | 메서드를 비동기로 실행하도록 표시 |
| @EnableAsync | 비동기 기능 활성화(프록시 생성) |
| TaskExecutor | 비동기 스레드를 관리하는 실행기 |
| ThreadPoolTaskExecutor | 실무에서 가장 많이 사용하는 스레드풀 구현체 |
| AsyncUncaughtExceptinoHandler | 비동기 메서드의 예외 처리 담당 |

- @EnableAsync 활성화
  - 비동기 기능을 사용하려면 먼저 @EnableAsync를 통해 Spring 컨테이너에 비동기 기능을 등록
  - 이 어노테이션은 내부적으로 AsyncAnnotationBeanPostProcessor를 등록
  - @Async가 붙은 메서드를 프록시로 감싸고 비동기 스레드 풀에 위임할수 있게 만듬.
 
- @Async
  - 비동기 처리를 선언적으로 구현할 수 있음
  - @EnableAsync가 활성화된 상태에서 @Async가 붙은 메서드는 별도의 스레드에서 실행되며, 호출자는 즉시 반환됨
  - 일반적으로 Service 계층의 메서드에 적용
  - 클래스 레벨에도 선언할 수 있음
    + 해당 클래스의 모든 public 메서드가 비동기로 동작
  - 지원 반환 타입
    + void : 단순 비동기 실행, 결과가 필요 없는 경우 / 알림, 로그, 통계 전송 등
    + Future<?> 또는 ListenableFuture<?> : 비동기 결과를 나중에 조회 가능 / 오래 걸리는 계산, 외부 API 요청
      + 결과를 동기적으로 가져올 수 있지만, 완료 여부를 직접 확인해야 함
    + CompletableFuture<?> : 비동기 완료 후 후속 작업(thenApply 등) 수행 가능 / 체이닝, 비동기 결합 처리
      + 비동기 결과를 체이닝 방식으로 다룰 수 있어 실무에서 가장 많이 사용 됨
      + 비동기 완료 후 thenApply, thenAccept 등을 사용해 후속 작업을 자연스럽게 연결할 수 있음.
      + 같은 클래스 내부에서 호출하면 비동기 처리가 되지 않음 (Self-Invocation 문제)
        + 비동기 동작시키려면 별도의 Bean에서 호출하거나, 프록시를 통해 호출해야 함
       
## TaskExecutor
- Spring 비동기 인프라의 핵심이며, 작업을 실행할 스레드를 관리하는 추상화 계층
  - 쓰는 이유
    - 프레임워크 전반의 일관성과 빈 기반의 설정/주입/관찰을 위해 TaskExecutor 추상화를 제공
    - 프레임워크 표준화 : 스케줄러, 이벤트, @Async등 여러 모듈이 같은 실행 추상화를 사용
    - 빈 기반 구성 : Yaml, Java Config로 환경별 다른 실행 전략을 쉽게 교체
    - 운영 가시성 : 스레드 이름 프리픽스, 메트릭 노출, 거부(포화) 정책 등 운영 친화 옵션과의 결합이 쉬움
    - 일관된 예외 처리 관점 : @Async와 결합 시 비동기 예외 처리 진입점이 명확해 짐
   
| 비교 항목 | ExecutorService | TaskExecutor |
|-|-|-|
| 정의 위치 | java.util.concurrent | org.springframework.core.task |
| 목적 | 스레드 풀 관리 및 실행 | Spring 비동기 작업 통합 관리 |
| 라이프사이클 | 수동 관리 (shutdown 필요) | Spring이 자동 관리(Bean 기반) |
| 예외 처리 | 직접 try-catch | AsyncUncaughtExceptionHandler 등 제공 |
| 통합성 | 독립적으로 동작 | @Async, @Scheuled와 자연스럽게 통합 |

- SimpleAsyncTaskExecutor
  - 매번 새로운 스레드 생성
  - 스레드 재사용이 없음
  - 대량 트래픽 상황에서 비효율적으로 거의 사용하지 않음.
- ThreadPoolTaskExecutor
  + 가장 일반적으로 사용하는 구현체
  + 내부적으로 Java의 ThreadPoolExecutor를 감싸서, 스레드 풀 재사용과 큐 관리 기능 제공
- ConcurrentTaskExecutor
  + Java의 ExecutorService를 직접 감싸서 제공하는 구현체
 
#### ThreadPoolTaskExecutor
- 비동기 실행을 위한 스레프풀 기반 실행기
- 내부적으로 Java의 ThreadPoolExecutor를 감싸고 있음
- 스프링 환경에 맞게 설정, 관리, 모니터링 기능 제공
- 스레드 풀 : 여러 스레드를 미리 생성해두고 재사용하는 구조
- 작업 큐 : 대기 중인 비동기 작업을 보관하는 공간
- 스레드 관리 정책 : 코어 스레드 유지, 최대 스레드 제한, 큐 용량 조절 등
- CorePoolSize : 항상 유지되는 기본 스레드 수
- MaxPoolSize : 동시 실행 가능한 최대 스레드 수
- QueueCapacity : 대기할 수 있는 작업의 최대 개수
- KeepAliveSeconds : 유휴 스레드가 종료되기 전 대기 시간
- allowCoreThreadTimeOut : 코어 스레드의 종료 허용 여부
- ThreadNamePrefix : 생성되는 스레드 이름 접두사 / 로그.모니터링 시 출처 식별에 유용

- rejectedExecutionHandler : 큐 초과 시 대체 동작 정의 / CallerRunsPolicy(안전), Default = AbortPolicy(빠른 실행)
  + DiscardPolicy 뒤 작업 다 버림 / DiscardOldestPolicy 가장 오래된 작업 버림
 
- 여러 TaskExecutor 활용 전략
  + CPU 연산 작업과 I/O 중심 작업을 같은 스레드 풀에서 처리하면 병목이 발생할 수 있음
  + 따라서 여러 TaskExecutor를 정의해 자원을 분리하는 것이 좋음

## Spring Event
- 애플리케이션 내부에서 비동기적으로 로직을 분리하고 결합도를 낮추기 위한 구조를 제공
- 이벤트를 발행하면, 등록된 리스너들이 이를 수신하여 후속 작업을 처리
- 구조적 개념
  + ApplicationEvent : 이벤트 객체 / 발생한 사건(이벤트)의 정보를 담는 객체
  + ApplicationEventPublisher : 이벤트 발행자 / 특정 시점에 이벤트를 발생시키는 역할
  + ApplicationListener : 이벤트 수신자 / 발행된 이벤트를 감지하고, 후속 동작 수행
 
- 장점 :
  + 결합도 감소와 유지보수성 향상
  + 확장성과 유연성
    + 새로운 기능을 추가해야 하는 경우, 기존 로직을 수정하지 않고 새로운 리스너만 등록하면 됨
  + 비동기 확장 기반
    + 이벤트 시스템은 이후 @Async와 결합하여 비동기 처리를 구현할 수 있음
    + 서비스의 응답 속도를 높이고, 부하를 분산시킬 수 있음
   
- @EventListener
  - condition 속성을 사용하여 특정 조건일 때만 이벤트를 처리할 수 있음
    + 이벤트의 필드 값을 검사해 필터링된 이벤트 처리가 가능
  - @EventListener(condition = "...") 내부에서 SpEL을 활용하면 이벤트 속성을 기반으로 정교한 조건식을 만들 수 있음
 
- @Async
  - @EventListener에 @Async를 함께 적용하면 이벤트 리스너를 비동기적으로 실행하게 만들어 줌.
 
- 이벤트 기반 비동기 처리 활용
  + 사용자 활동 로깅
  + 알림 발송 시스템
  + 캐시 갱신 처리
  + 외부 시스템 연동
 
- Spring Event 동작 범위
  + ApplicationContext 내부에서만 작동
  + 이벤트를 발행한 인스턴스(JVM) 안에서만 수신 가능
  + 문제 예시
    + 회원가입 후 이메일 발송 : 요청을 처리한 인스턴스에서만 이벤트 발생 -> 이메일 누락 가능
    + 주문 생성 후 결제 서비스 호출 : 주문 서비스에서만 이벤트가 전달됨 -> 결제 서비스와 데이터 불일치
    + 서버 확장 (Auto Scaling) : 인스턴스 간 이벤트 전달 불가 -> 데이터 일관성 깨짐
   
- 메시지 브로커
  - 여러 애플리케이션 간에 메시지(이벤트)를 중간에서 안전하게 전달해주는 시스템
  - Kafka / RabbitMQ / amazon SQS, google Pub/Sub 등
  - Producer : 메시지를 발생하는 주체 (Publisher 역할)
  - Broker : 메시지를 임시 저장 및 관리하는 서버
  - Topic/Queue : 메시지를 구분하기 위한 논리적 공간
  - Consumer : 메시지를 구독하여 처리하는 주체
 
- 도메인 이벤트 vs 통합 이벤트
  - **도메인 이벤트**
    - 하나의 서비스 내부에서 발생하는 비즈니스 상태 변화를 표현
    - 이벤트를 통해 내부 모듈 간 결합도를 낮춤
    - @EventListener 기반으로 비동기 처리할 수 있음
  - **통합 이벤트**
    - 서비스 간 데이터 교환을 위한 이벤트
    - 다른 애플리케이션이 구독할 수 있도록 외부 브로커에 발행 됨

## 비동기 예외 처리
- 비동기 메서드는 새로운 스레드에서 실행
- 메인 스레드에서 발생하는 예외 전파 규칙이 그대로 적용되지 않음
- 비동기 메서드는 호출 즉시 반환되기 때문에, 작업의 성공 여부나 실패 여부를 즉시 확인할 수 없음.

- 각 스레드는 독립적인 호출 스택(Call Stack)을 가지고 있으며
  - 비동기로 실행되는 메서드는 완전히 다른 스레드의 스택 공간에서 동작.
  - 한쪽 스레드의 예외는 다른 쪽으로 전파되지 않음.

- @Async는 내부적으로 ExecutorService를 사용해 새로운 스레드를 생성하고, 그 결과를 Future 객체로 래핑하여 관리
  + 이 구조 덕분에 비동기 처리가 가능
  + 반환 타입이 void인 경우에는 Future가 생성되지 않기 때문에 결과나 예외를 추적할 방법이 없음
  + 따라서 비동기 결과 추적이 필요한 경우 반드시 Future나 CompletableFuture를 반환해야 함.
 
- 비동기 환경에서는 일반적인 try-catch 대신 비동기 스레드의 예외를 별도로 수집하고 처리하는 전략이 필요
- 로그는 스레드 이름 필수

- **AsyncUncaughtExceptionHandler**
  - 비동기 메서드는 호출 스레드와 별개로 실행되므로, 예외가 발생해도 호출자에게 전파되지 않음
  - 이때 비동기 스레드 내부에서 발생한 예외를 감지하고 로깅하기 위해 사용하는 것
  - @Async로 실행된 void 반환 메서드의 예외를 감지하기 위한 전용 인터페이스
  - 비동기 스레드에서 예외가 발생하면, 스프링이 handleUncaughtException()을 자동으로 호출
    + 로그에는 예외 발생 위치, 타입, 인자 값이 기록 됨
  - void 메서드에서만 작동함
    + Future, completableFuture 기반 메서드는 반환 객체 자체에서 예외를 감지

- **예외처리**
  + 단순히 try-catch로 감싸는 것 만으로는 충분하지 않음
  + Retry
    + 일시적인 네트워크 오류나 타임아웃 등의 문제로 작업이 실패 했을 시
    + 동일 작업을 일정 횟수 다시 시도하는 전략
    + 일시적인 장애를 복구하는데 효과적
      + 최대 시도 횟수 + 대기 간격을 함께 설정
  + Fallback
    + 특정 작업이 실패했을 때 대체 동작
    + 시스템의 가용성을 유지하는 방법
    + ex: 외부 이메일 서버가 작동하지 않을 때 임시 메시지 큐에 저장하거나 관리자에게 알림을 전송할 수 있음
  + Circuit Breaker (회로 차단기)
    + 외부 시스템에 지속적으로 실패 요청이 발생할 경우, 일정 시간 동안 호출을 중단하여 시스템을 보호하는 패턴
    + Closed : 정상 상태, 모든 요청 허용
    + Open : 오류율이 일정 임계치 초과 -> 호출 차단
    + Half-Open : 일부 요청 허용 -> 성공 시 복구, 실패 시 다시 차단
    + Normally Closed Switch 와 비슷함 -> 평상 시 정상 / 동작 시 off
   
## TaskDecorator
- 비동기 작업은 별도의 스레드에서 실행되므로, 기존 요청 스레드의 ThreadLocal 변수가 그대로 전달않음
- 이로 인해 보안 컨텍스트, 트랜잭션 정보, 로깅 컨텍스트(MDC) 등이 손실되는 문제가 발생
- ThreadLocal
  - 각 스레드마다 독립적인 데이터를 저장할 수 있도록 도와주는 클래스
  - 여러 스레드가 동시에 하나의 객체를 공유하더라도 TreadLocal을 사용하면 각 스레드는 자신만의 복사본을 가지게 됨
    + 스레드마다 별도의 저장소를 가지므로 동시성 문제를 방지할 수 있음
    + 하지만 새로운 스레드가 생성되면 기존 스레드의 ThreadLocal 데이터는 자동으로 복사되지 않음
   
- 실행 컨텍스트 손실
  + 로그 추적 컨텍스트(MDC) 손실
    + 로그 추적 불가
    + 보안 컨텍스트 손실
    + 트랜잭션 단절
   
- MDC
  - 로그 추적을 위한 ThreadLocal기반의 데이터 저장소
  - 로그백이나 Log4j에서 제공
 
- 해결 방향 - TaskDecorator
  - 비동기 작업이 실행되기 직전에 스레드 컨텍스트를 복사하여 새로운 스레드에도 동일한 환경을 적용할 수 있도록 해주는 장치
  - 비동기 실행 시 TaskDecorator가 기존 스레드의 ThreadLocal 데이터를 복사하여 새 스레드에 전달
  - @Async, ExecutorService, ThreadPoolTaskExecutor 어디서나 적용 가능
  - 스프링 환경에서는 ThreadPoolTaskExecutor#setTaskDecorator() 로 등록 가능
 
#### TaskDecorator란
- 비동기 작업(Runnable)이 실행되지 전에 실행 환경을 래핑 하여 커스터마이징할 수 있도록 설계됨
  + 실행할 작업(Runnable)을 받아, 새로운 Runnable로 감싸서 반환하는 인터페이스
- 실행 전/후에 커스텀 로직 주입 가능
- 단순 로깅보다 ThreadLocal 데이터(MDC, SecurityContext 등) 을 복사하는데 활용
  + `MDC.getCopyOfContextMap()` 현재 스레드의 로그 컨텍스트를 복사
  + `MDC.setContextMap()` 으로 동일한 값 설정
  + finally 블록에서 MDC.clear()를 반드시 호출해, 스레드 재사용 시 이전 데이터가 남지 않도록
- 장점 :
  + 유연성 : 모든 비동기 실행 전후에 커스텀 로직 추가 가능
  + 재사용성 : 여러 Executor에서 동일한 데코레이터 사용 가능
  + 일관성 : 공통된 ThreadLocal, 로깅, 보안, 트랜잭션 관리 가능
  + 안정성 : 실행 후 정리(Clean-up)을 통한 메모리 누수 방지
- 두 개 이상의 데코레이터 적용
  - 두 데코레이터를 조합하는 별도의 CompositeTaskDecorator를 만들어야 함
 
- 트랜잭션 컨텍스트 관리
  - 비동기 스레드에서는 TransactionSynchronizationManager가 초기화되지 않아 새로운 트랜잭션 처럼 동작
    + 트랜잭션이 전혀 존재하지 않는 상태가 될 수 있음
  - TranscationSynchronizationManager
    + 현재 스레드의 트랜잭션 관련 정보를 관리하는 스프링의 내부 유틸리티 클래스
    + 트랜잭션 매니저가 이 객체를 통해 리소스 바인딩, 트랜잭션 상태, 동기화 콜백 등을 관리
    + getResourceMap()
    + bindResource()
    + unbindResource()
    + isActualTransactionActive()
   
## WebClient
- Spring WebFlux에서 제공하는 비동기-논블로킹 HTTP 클라이언트
  - 기존 RestTemplate의 단점을 보완하여, Reactive Streams 기반의 비동기 처리 모델을 제공
 
| 비교 항목 | RestTemplate | WebClient |
|-|-|-|
| 통신 방식 | 동기 | 비동기 |
| 동작 방식 | 요청-응답 완료 시까지 스레드 대기 | 요청 후 스레드 반납, 응답 시 콜백 처리 |
| 스레드 사용량 | 요청당 1개 스레드 고정 | Event Loop 기반, 적은 스레드로 다수 요청 처리 |
| 지원 기술 | Spring MVC (Tomcat) | Spring WebFlux(Reactor 기반) (Netty) |
| 권장 여부 | Deprecated | 공식 권장 대체 기술 |

- 비동기 : 요청을 보낸 후 결과를 기다리지 않고 다음 작업을 수행
- 논블로킹 : I/O 작업 중 스레드가 대기하지 않음

- Reactive Streams 기본 개념
  - WebClient 는 Reactive Streams API (Publisher, Subscriber, Subscription)를 기반으로 동작
  - Publisher : 데이터를 발행하는 쪽 (WebClient)
  - Subscriber : 데이터를 구독하고 처리하는 쪽 (응답 처리자)
  - Subscription : Publisher <> Subscriber 간의 데이터 흐름 제어 (BackPressure)
 
- 사용되는 이유
  - 외부 API 연동 : 외부 서버 응답을 기다리는 동안 스레드 낭비 없이 처리 가능
  - 대량 비동기 호출
  - 마이크로서비스 간 통신 : 서비스간 REST 호출을 효율적으로 수행
  - 파일 업로드/다운로드 : 대용량 I/O를 논블로킹으로 처리 가능
 
- WebClient.Builder
  + baseUrl() 모든 요청 공통 URL prefix 지정
  + defaultHeader(), defaultCookie() : 모든 요청에 기본 헤더나 쿠키를 자동으로 포함
  + filter() : 요청 또는 응답을 가로채어 로깅, 인증, 예외 처리 등을 수행
    + ExchangeFilterFunction 인터페이스를 이용하여 구현
    + 모든 요청/응답을 가로채어 공통 관심사 처리
  + exchangeStrategies()
    + WenClient가 내부적으로 사용하는 데이터 인코딩/디코딩 전략, 메시지 변환기, 메모리 버퍼 관리 정책을 제어하는 고급 설정
    + 대용량 데이터를 처리하거나, 커스텀 인코더/디코더를 등록할 때 반드시 알아두어야 함
  + 

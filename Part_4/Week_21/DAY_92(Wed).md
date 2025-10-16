# DAY_92

## 목차
- [메서드 보안](#메서드-보안)
- 
---

## 메서드 보안
- JWT 인증이 완료되면, 이후 단계에서 사용자의 권한이나 소유 관계를 기반으로 접근을 제어해야 함.
- URL 기반 접근 제어 `.antMatchers`는 단순하지만, 비즈니스 로직 내부에서 더 세밀한 제어가 필요할 때는 한계가 있음.
- 따라서 `@PreAuthorize`를 사용해 메서드 단위로 접근 권한을 정의
- `@PreAuthorize("hasRole('ADMIN')")`
- `@PreAuthorize("#memberId == authentication.principal.memberId")` 처럼 SpEL로 표현 가능
- `@EnableMethodSecurity`가 활성화 되어야 함.
  - SecurityConfiguration에 적용
- JWT에 포함된 클레임(ex: memberId, roles, organization)을 활용하여 추가 검증 가능
- ```java
  @PreAuthorize("#memberId == authentication.principal.claims['memberId'] or hasRole('ADMIN')")
  public void updateMember(Long memberId) {
      // 본인 혹은 관리자만 수정 가능
  }
  ```
- 리소스의 주인만 접근할 수 있도록 하는 로직을 서비스 계층에 추가할 수 있다. ( 커스텀 인가 표현식 가능 )
- ```java
  @Service
  @RequiredArgsConstructor
  public class OrderService {
  
      private final OrderRepository orderRepository;
  
      @PreAuthorize("@permissionEvaluator.isOrderOwner(#orderId, authentication)")
      public void cancelOrder(Long orderId) {
          Order order = orderRepository.findById(orderId)
              .orElseThrow(() -> new BusinessLogicException(ExceptionCode.ORDER_NOT_FOUND));
          order.cancel();
      }
  }
  ```
  ```java
  @Component("permissionEvaluator")
  @RequiredArgsConstructor
  public class CustomPermissionEvaluator {
  
      private final OrderRepository orderRepository;
  
      public boolean isOrderOwner(Long orderId, Authentication authentication) {
          Long memberId = ((MemberDetails) authentication.getPrincipal()).getMemberId();
          return orderRepository.findById(orderId)
              .map(order -> order.getMember().getMemberId().equals(memberId))
              .orElse(false);
      }
  }
  ```

## Refresh Token
- Token은 발급 이후 서버에서 관리가 불가능함.
- Access Token은 탈취 되면 위험하기 때문에 만료시간을 짧게 설정함
- 만료 시간이 짧아 사용자가 자주 로그인 해야 하고 불편한 사용자 경험을 줌.
- 만료시간이 긴 RefreshToken을 이용하여 AccessToken을 재발급 함.
- 서버 DB 혹은 안전한 스토리지에 보관함.

#### 토큰 회전 전략
- Refresh Token이 재발급될 때마다 이전 토큰을 무효화하고 새로운 토큰으로 교체하는 방식
- AccessToken 재발급 시 RefreshToken도 재발급

| 주제         | 권장 방식                           |
|-------------|----------------------------------|
| 토큰 저장소    | RDB 또는 redis 사용                 |
| 토큰 암호화    | 평문 저장 금지, SHA256 해싱 권장       |
| 만료 시간 관리 | LocalDateTime 또는 Instant로 관리    |
| 회전 전략     | 새 토큰 발급 시 기존 토큰 즉시 무효화      |
| 로그아웃 처리  | invalidateToken() 호출로 서버에서 삭제 |


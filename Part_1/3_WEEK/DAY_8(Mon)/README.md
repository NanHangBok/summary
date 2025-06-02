## DAY_8(Mon)

- 로그 상 채널을 생성할 때 addChannel 메서드에서 NPE(NullPointException) 발생
- 이전에 JCFMessageService.getInstance() 전까지는 문제가 없었음 -> JCFMessageService.getInstance() 메서드가 문제라고 판단
- 확인해보니 main에서 User와 Channel 이 MessageService 인스턴스를 생성하기 전에 MessageService 인스턴스를 호출 중
- -> null 객체가 저장됨 -> 이후 JCFMessageService를 호출 할 시 null 객체가 호출되며 NPE 발생
- UserService와 ChannelService가 MessageService가 인스턴스화 되기 전에 호출 하지 않도록 method에 필요할 때 마다 인스턴스를 가져오도록 변경
- 이후 NPE 해결

- FEEDBACK ( FACTORY 패턴 구현 )
  - 팩토리 패턴 구현 성공
  - 떨어져 있던 추가와 수정 삭제 기능을 엔티티에서 진행하는 것으로 합침
  - 반복적으로 부르던 메서드를 한번에 진행하게 코드 수정
    - 그 결과 리스트가 반복하며 CONCURRENTMODIFICATIONEXCEPTION 발생
    - 메서드를 반복적으로 콜 하는 것에서 그냥 한번 삭제하는걸로 수정
    - ```
      public void deleteUser(User user) {
        if (users.contains(user)) {
            users.remove(user);
            user.deleteChannel(this);
            setUpdatedAt(System.currentTimeMillis());
        }
      }
      -->
      public void deleteUser(User user) {
        if (users.contains(user)) {
            users.remove(user);
            user.getChannels().remove(this);
            setUpdatedAt(System.currentTimeMillis());
        }
      }
      ```

## DAY_8(Mon)

- 로그 상 채널을 생성할 때 addChannel 메서드에서 NPE(NullPointException) 발생
- 이전에 JCFMessageService.getInstance() 전까지는 문제가 없었음 -> JCFMessageService.getInstance() 메서드가 문제라고 판단
- 확인해보니 main에서 User와 Channel 이 MessageService 인스턴스를 생성하기 전에 MessageService 인스턴스를 호출 중
- -> null 객체가 저장됨 -> 이후 JCFMessageService를 호출 할 시 null 객체가 호출되며 NPE 발생
- UserService와 ChannelService가 MessageService가 인스턴스화 되기 전에 호출 하지 않도록 method에 필요할 때 마다 인스턴스를 가져오도록 변경
- 이후 NPE 해결


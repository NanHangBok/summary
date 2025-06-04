## DAY_10

지금까지의 피드백

/****************************************
         *  외부에서 Factory 금지  / COMPLETE
         *  user와 userid 사용하는거 통일 시키기  / COMPLETE
         *  메서드 이름 바꾸기 deleteChannelUser  / COMPLETE..?
         *  isActive enum으로 구현하기  / COMPLETE -> status로 필드명 변경
         *  update select말고 전체를 입력받아서 수정된 부분만 수정하기 혹은 enum 사용하기  / COMPLETE + enums 패키지 추가
         *  Factory 폴더 따로 빼기 / COMPLETE
         *  엔티티 공통 필드 basedEntity로 따로 빼기 ( id, createdAt, UpdatedAt ) / COMPLETE
         *  setUpdated는 Service에서 / COMPLETE
         *  -
         *  updateField 사용하지 않기  / COMPLETE
         *  status getter 생성자 ONLINE("온라인") = value  / COMPLETE
         *  JCF 최대한 서비스에서 안 불러오기  / MessageService가 채널과 유저에 의존하지 않게(채널 서비스와 유저 서비스에 접근하지 않도록) 수정
         *  -
         *  stream().map으로 수정  / COMPLETE
         *  channel과 유저에서 addMessage와 removeMessage를 메세지와 상호 보완적으로 수정  / COMPLETE
         *  updateUser 몇 개의 인자가 오든 대응 가능하게 수정 / COMPLETE
****************************************/

### 회고
##### 1
채널 삭제 시 - 메시지 리포지토리에서 채널에서 작성된 메시지를 삭제해야 하기 때문에 메시지 서비스 참조가 필요 <br>
유저 삭제 시 - 메시지 리포지토리에서 유저가 작성한 메시지를 삭제해야 하기 때문에 메시지 서비스의 참조가 필요 <br>

메시지 삭제 시 - 메시지는 userID와 channelID만 가지고 있기 때문에 유저 서비스와 채널 서비스에 접근하여 User와 Channel을 가져와야 함 <br>

이렇게 순환참조가 발생하는데 생성자를 통해 주입할 시 <br>
this.jcfMessageService is null이라며 NPE가 발생 <br>
> 메시지가 userId와 channelId를 가짐
>> User와 Channel을 필드로 가질 시 필요 이상의 정보를 가질 것 같음
>> 리포지토리를 사용하지 않기 때문에 순환참조 해결이 어려움
>>> 메시지가 User와 Channel을 객체로 가지도록 변경
> 이렇게 UserService, ChannelService, MessageService가 서로를 의존하지 않도록 코드가 수정 됨.
>> MessageService는 UserService와 ChannelService를 참조하지 않고
>> UserService와 ChannelService는 MessageService를 참조하도록 변경

##### 2
'''
public void removeChannel(Channel channel) {
        if(channels.contains(channel)) {
            channels.remove(channel);
            channel.removeUser(this);
        }
    }
로
.forEach(user -> user.removeChannel(channel));
이거로 하면 concurentModificationException이 발생
.forEach(user -> user.getChannels().remove(channel))로 수정
'''
위에 경우 channels를 참조하는 와중에 remove를 통해 채널을 삭제해서 ConcurentModificationException 발생
아래의 경우 Channels를 얻고 삭제하기 때문에 참조가 끝나고 channel을 삭제하게 됨
- ConcurentModificationException 발생 X
> ConcurentModificationException 스트림 순회 중 컬렉션의 구조 변경 시 발생하는 Exception





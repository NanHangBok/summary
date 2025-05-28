# WEEKLY_PAPER
- [2주차](#2주차) [rebase와 merge / fetch와 pull]
- [3주차](#3주차)

## 2주차

+ git rebase와 git merge의 차이점을 설명하고, 각각 어떤 상황에서 사용하는 것이 적절한지 설명해주세요.
  + **rebase**의 경우 히스토리가 남지 않고 **히스토리가 선형**이 되고 개인 브랜치에서 추가 브랜치를 만들어 개인적으로 사용할 때 보통 사용하고,
  + **merge**의 경우 히스토리가 남는다.
  + *( 개인적인 생각으로 기업에서 버전관리를 중요시한다면 불필요한 경우 merge를 사용하여 히스토리를 남겨 버전 업데이트 내역에 대한 가독성을 떨어트리는 것 보다 rebase를 통해 히스토리를 남기지 않고 병합하는 방식이 적합할 것 같다.)*


+ git fetch와 git pull의 차이점을 설명하고, 각각을 사용하는 것이 적절한 상황을 설명해주세요.
  + fetch의 경우 **local repository로 데이터를 가져오기** 때문에 working directory를 건들지 않기 때문에 현재 작업중인 내용이 수정되지 않기 때문에 변경사항을 확인할 때 사용할 수 있고 
  + pull의 경우 **remote repository에서 working directory로** 가져오기 때문에 자신의 **working directroy(local repository)와 remote repository를 연결**할 수 있고 현재 working directory의 **작업이 덮어 씌워지며** 작업중인 것이 유실 될 수 있다.
---
## 3주차

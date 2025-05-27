### 2주차
<hr>
+ git rebase와 git merge의 차이점을 설명하고, 각각 어떤 상황에서 사용하는 것이 적절한지 설명해주세요.
  ++ rebase의 경우 히스토리가 남지 않고 개인 브랜치에서 추가 브랜치를 만들어 개인적으로 사용할 때 보통 사용하고, merge의 경우 히스토리가 남는다.
+ git fetch와 git pull의 차이점을 설명하고, 각각을 사용하는 것이 적절한 상황을 설명해주세요.
  ++ fetch의 경우 local repository로 데이터를 가져오기 때문에 working directory를 건들지 않기 때문에 현재 작업중인 내용이 수정되지 않기 때문에 변경사항을 확인할 때 사용할 수 있고 
  ++ pull의 경우 remote repository에서 working directory로 가져오기 때문에 자신의 working directroy(local repository)와 remote repository를 연결할 수 있고 현재 working directory의 작업이 덮어 씌워지며 작업중인 것이 유실 될 수 있다.

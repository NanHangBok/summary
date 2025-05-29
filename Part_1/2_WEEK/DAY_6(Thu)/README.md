# DAY_6

- 디스코드 시스템 콘솔기반 구현
  - 클래스 내부에 List<클래스> 구현
  - toString 진행 시 List내부의 클래스를 호출하고 해당 클래스 내부에 호출한 클래스가 들어있는 List가 있음
  - toString이 순환하며 스택오버플로우 발생
  - -
  - toString을 List<클래스>가 아닌
  - stream을 통해 toList()로 String타입으로 재생성
  - toString에 List<String> 반환으로 수정 -> 순환구조 해소 및 스택오버플로우 해결

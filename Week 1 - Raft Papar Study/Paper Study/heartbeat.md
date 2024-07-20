--- 
###### 하트 비트
- leader -> follower 일정 시간 간격, 반복 전달 메시지
- 클라이언트의 명령 전파를 위한 log X
- 리더가 자신의 상태를 유지하는 수단
- 빈 로그 엔트리를 가진 `AppendEntries` [[AppendEntries.md|RPC]]로 구현
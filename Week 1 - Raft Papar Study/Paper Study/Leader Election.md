Leader를 새롭게 선정하는 과정

Follower는 [[Election timeout]] 동안 [[heartbeat]]를 받지 않는다면 Candidate가 되고, Leader Election이 실행된다

## Procedure
1. Follower가 Election timeout동안 heartbeat를 받지 않음
2. Follower 는 Candidate로 변경됨
3. Candidate가 된후 자신의 Term을 +1 시킴
4. VotedFor 에 자신의 ID를 넣음
5. 시스템 내 다른 노드에게 [[RequestVote]] 메세지를 전달
6. 결과
	- 과반수 이상의 노드로부터 granted Vote를 받은 경우, Candidate는 Leader가 됨
		- 자신의 Term보다 낮은 Term을 갖는 응답메세지는 무시
	- 다른 노드가 Leader가 된 경우, 자신의 Term 이상의 heartbeat를 받으면서 Candidate는 Follower가 됨
		- 자신의 Term보다 낮은 Leader Term이면 무시
	- 일정기간 동안 Leader가 선출되지 않은 경우, 3번과정부터 반복
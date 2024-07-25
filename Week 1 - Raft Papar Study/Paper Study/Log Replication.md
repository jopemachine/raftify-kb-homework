Follower 들의 log를 Leader log와 일치시키는 과정.

## Procedure
### Leader
1. Client로부터 요청을 전달받음
2. Client로부터 전달받은 요청을 Log에 추가
3. 각 Follower 에 병렬적으로 로그 복사 요청 메세지 전송 ([[AppendEntries]])
	- 일정 기간동안 Follower로부터 응답받지 못한다면 로그 복사 요청 메세지 재전송
4. 과반수 이상의 Follower 가 로그 복사가 완료된다면, 추가된 log entry를 state machine에 적용
	- Commited Entry: 적용된 log entry
	- 
1. Commited Index를 Follower에게 병렬적으로 전달


이전 Term의 log entry가 과반수 node에 복사되었더라도 commit되었음을 보장할 수 없다. 왜냐하면 이전 Term의 Leader가 Commit하기 이전에 장애가 발생할 수 있기 때문이다.
만약 Leader가 이전 Term의 log entry가 과반수가 되었을 때 해당 log entry를 commit한다고 가정하자. 이 경우 Log가 불일치할 수 있고, [[Safety Machine Property]]를 위반할 수 있다.
Example)
![[Pasted image 20240725230804.png]]
Term 2 (a): S1이 Leader가 됨. Term2 log가 S2에 복사되다가 S1 Crash
Term 3 (b): S5가 Leader가 됨 (S3, S4 투표). Term3 log가 생성되지만 복사되지 않고 S5 Crash
Term 4 (c): S1이 Leader가 됨 (S2, S3, S4 투표). Term4 log가 생성되지만 Term2 log가 S2, S3에 복사되고 S1 Crash 

만약 Leader가 이전 Term의 log entry로 commit할 수 있다고 하자.

Term 5 (d): S5가 리더가 됨 (S2, S3, S4 투표 - S2, S3의 last entry의 term이 S5보다 작으므로 S5를 투표).
만약 S1에서 Term2 log가 과반수가 되었고, 이를 commit했다고 하자. 그리고 commit 정보를 S2까지 전달했다고 하자.
S5는 Term3 log를 S4에 복사하고, S3도 아직 Term2 log가 commit됨을 알지 못하기 때문에 이를 버리고 Term3 log를 복사한다.
이 경우 어떤 node는 state machine에 Term2 log를 적용하고, 어떤 node는 state machine에 Term3 log를 적용해 Safety Machine Property를 위반하게 된다.

이러한 문제를 피하기 위해서 Leader는 이전 Term의 log entry가 과반수가 되어도 이를 commit하지 않는다. 오로지 현재 term의 log entry가 과반수가 되었을 때 commit을 실행한다.
이럴 경우, (d)에서 Term2 log가 commit되지 않기 때문에 문제가 발생하지 않는다.
--- 
##### 상태 (States)
![[Pasted image 20240720112112.png]]
###### 모든 서버들의 persisted 상태
- 아래 상태들은 RPC에 응답하기 전 [[Stable Store.md|stable store]]에 업데이트
	- `currentTerm`: 서버가 관측했던 가장 최신의 `term`값
	- `votedFor`: `currentTerm`에서 투표한 candinate의 Id. 없는 경우 _null_.
	- `log[]`: 로그 엔트리들의 배열. 각 로그 엔트리들은 [[State Machine.md|state machine]]을 위한 명령&엔트리가 leader에게 수신 되었을 때의 `term` 값 포함
###### 모든 서버들의 휘발성(volatile) 상태
- `commitIndex`: commit 된 것으로 알려진 로그 엔트리들 중 가장 높은 `index` 값
- `lastApplied`: [[State Machine.md|state machine]]에 적용된 로그들 중 가장 높은 `index` 값
###### leader의 휘발성(volatile) 상태
- 아래 상태들은 leader election 이후 초기화 
	- `nextIndex[]`: 각각의 서버들에 대해 다음에 보내줘야 하는 로그 엔트리의 `index` 값들
	- `matchIndex[]`: 각각의 서버들에 대해 복제된 것으로 알려진 가장 높은 로그 엔트리의 `index` 값들
###### 모든 서버들
- `commitIndex` > `lastAppied`인 경우, `lastApplied`↑,  [[State Machine.md|state machine]]에 `log[lastApplied]` 적용
- RPC 요청이나 응답에 포함된 `term` 값이 `currentTerm`보다 높은 경우, `currentTerm`을 `term` 값으로 설정, follower 상태로 복귀
###### Leader\
- cluster을 대표하는 단일 노드
- leader는 클라이언트 요청의 수신 및 전파, 그리고 응답
- 로컬에 log 적재, 모든 follwer에게 전달
- election 후
	- 각 서버들에게 비어 있는 `AppendEntries` RPC 호출
	- Idle일 동안 `Election timeout` 막기 위해 지속적으로 heartbeat 전송
- 클라이언트 명령
	- 엔트리를 로컬 log에 추가
	- 상태 머신에 엔트리를 적용 -> 응답
- `nextIndex[]` 배열을 순회 -> `last log index >= nextIndex`인 follower들에 대해 `nextIndex`에서 시작하는 로그 엔트리들의 배열을 `entries` 인자에 넘겨 `AppendEntries` RPC 호출
	- 성공하는 경우 해당 follower의 `nextIndex`, `matchIndex`를 업데이트
	- `AppendEntries`가 로그 비일관성으로 인해 실패한다면 `nextIndex`를 낮추고 Retry 요청
- `N > commitIndex`, 대다수 서버들(_Majority_)에 대해 `matchIndex >= N`, `log[N].term == currentTerm`을 만족하는 `N`이 존재하는 경우, 대입
###### Follower
- 클라이언트로 받은 요청을 leader로 redirect 
- leader의 요청을 수신 처리
- candinate와 leader들의 RPC에 응답
- `Election timeout` 시간 동안 leader와 candinate에게 `AppendEntries` 요청을 받지 못한 경우 candinate가 됨
###### Candidate
- 새 leader를 선출하기 위해 candinate가 된 상태
- leader로부터 일정 시간 이상 [[heartbeat]] 받지 못한 follower -> candinate
- candinate 전환 -> leader election을 시작
    1. `currentTerm` 증가
    2. 자기 자신에게 투표
    3. `Election timer` 초기화
    4. 다른 follower에게 [[RequestVote.md|RequestVote]] RPC 전송
- 다수의 서버가 vote -> leader
- leader -> `AppendEntries` RPC -> candiante들은 follower가 됨
- `Election timer` 만료 -> 새 투표 시작
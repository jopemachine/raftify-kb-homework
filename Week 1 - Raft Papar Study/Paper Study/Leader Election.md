--- 
###### 리더 선출
- Raft는 [[heartbeat]]를 통해 리더 선출
- 서버-> follower [[Server State.md|상태]] 시작
- follower는 leader or candiante에게 유효한 [[AppendEntries.md|RPC]]받음 -> 상태 유지
- leader -> follower 주기적으로 [[heartbeat]] 전파
- leader가 `Election timeout`동안 [[AppendEntries.md|RPC]] X -> leader 장애 간주 -> new leader election 시작

- 선거 시작을 위해 follower `current term`↑ -> candinate
- 자신에게 투표 -> `RequestVote` RPC를 클러스터 내 서버들에게 병렬 실행
- candinate는 아래 세 가지 중 하나가 일어날 때까지 candinate 유지
	1. 내가 투표에서 이겨서 leader가 되거나
	2. 다른 candinate가 leader가 되거나
	3. [[Election timeout.md|Election timeout]]이 됐는데 leader X

- candinate -> 클러스터 내 같은 [[term.md|term]] 값을 가진 다수(_Majority_)에게 [[RequestVote.md|투표]] -> leader
- 리더로 선출된 직후 추가적인 선거가 일어나는 것을 방지하기 위해 다른 모든 서버들에게 [[heartbeat.md|heartbeat]] 전송

- 투표 결과를 기다리는 동안 candinate는 leader라고 주장하는 서버로부터 `AppendEntries` RPC를 받을 수도 있음
	- leader의 `term` 값이 구식 -> RPC 거절
	- leader의 `term` 값이 최신 -> 클러스터 내부에서 리더 선출 성공 -> candinate가 follower로 돌아감

- follower들이 다들 candinate로 참가하면 표 분산 -> 어떤 서버도 leader X
- candinate들의 `RequestVote` RPC-> timeout ->`term` 값↑, 새 선거
	- [[Split Vote.md|표 분산]]이 계속 될 수 밖에 없음 -> 교착 상태
	- `Election timeout` 값을 (_150ms - 300ms_) 랜덤 선정해 해결
	- 어떤 서버가 먼저 투표에 승리 -> 다른 후보자들이 타임 아웃 되기 전에 heartbeat 보낼 수 있음
___ 
###### 요약
1. leader가 election timeout 전에 heartbeat 안 보내거나 term이 끝남
2. candinate가 되서 스스로에게 한 표 행사
3. 과반수 받는 candinate -> leader 되서 Append Entries RPC 뿌림
4. 다른 cadniate들은 새 leader의 Append Entries RPC를 받아서 current term을 확인
	1. 나보다 최신임 -> follower로 회귀
	2. 느린데? -> 씹음
5. 다시 leader은 계속 heartbeat 보내고 앞의 행위 반복
6. 만약 term동안 leader 못 뽑음
	- term 증가
	- 다시 leader 선출
7. 모든 follwer가 동시에 candinate가 되면?
	-  교착 상태 진입
	- 그러므로 election timeout 값은 follower마다 랜덤
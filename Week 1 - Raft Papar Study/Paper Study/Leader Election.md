Raft는 [[heartbeat]] 메커니즘을 사용하여 Leader Election (리더 선출) 한다. 서버가 시작되면, 모든 노드들은 Follower 상태로 시작한다.
Leader는 주기적으로 heartbeat를 메시지를 보내서 모든 follower들이 그들의 권위(authority)를 유지한다.
> heartbeat 메시지는 [[AppendEntries]] RPC로 로그 엔트리가 없다

[[Election timeout]]은 follower가 특정 시간이 지나도 통신을 못 받아서 새로운 Leader를 선출하기 위한 선거를 시작한다. 즉 여기서 follower들 중 제일 먼저 timeout에 도달하면 candidate가 된다.
Follower는 현재 term을 증가시키고 candidate 상태로 바뀐다. 자기 자신에게 투표를 하고 [[RequestVote]] RPC 요청을 cluster에 있는 다른 노드에게 요청한다.

Candidate는 다음 3가지 일이 발생할 때까지 candidate 상태를 유지한다.
- 선거에 승리함
- 특정 시간이 지나도 아무도 선출되지 않았음
- 다른 노드가 Leader로 선출됨

같은 term에서 cluster 내에 서버의 다수결로부터 투표를 얻게 된다면 candidate는 선거에 승리하면서 Leader가 된다. Leader가 된 노드는 다른 서버들에게 heartbeat 메시지를 보내 권위(authority)를 확인한다.
투표를 기다리는 동안, candidate는 다른 서버로부터 leader가 되기 위한 AppendEntries RPC 요청을 받을 수 있다. 만약 leader의 term이 candidate의 현재 term보다 크다면, candidate는 정당한 leader로 판단하고 follower 상태로 되돌아간다.
> term 정보는 RPC에 포함되어 있다

Candidate가 선거에 이기거나 지지 않는 상황이다. 몇몇의 follower들이 동시에 candidate이 된다면, 투표는 분할(split)되어 candidate는 다수결의 표를 얻지 못하게 된다. 이런 상황이 발생하게 된다면, candidate들은 term를 증가시키고 새로운 라운드(Round)의 RequestVote RPC를 시작한다. 그러나 추가적인 조치가 없으면 분할 투표([[Split Vote]])는 무한으로 발생하게 된다.

Raft는 랜덤의 election timeout을 사용하여 분할 투표가 드물게 일어나며 빠르게 해결된다. 분할 투표를 막기 위해, election timeout은 고정된 주기(150ms ~ 300ms)의 랜덤값으로 선택한다. 분할 투표가 발생할 때에도, 랜덤의 election timeout을 재시작하고 다음 투표를 시작한다.
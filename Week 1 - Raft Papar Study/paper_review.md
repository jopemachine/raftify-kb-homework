# Raft Algorithm

Raft Algorithm 은 분산 시스템 환경에서, replicated log 를 관리하기 위한 consensus algorithm 이다. Replicated State machine 이 있는 환경에서 사용한다. MySQL 에서 replica 서버를 보면 apply 된 이후의 데이터의 정합성을 맞추는 것이 아닌, query 를 로그처럼 관리하고, 이를 보내 replica 서버에서 CUD 를 통해 정합성을 맞추는데, 그 용도와 동일하다. 로그를 replicated 하여 관리하여 state machine 을 즉각 동일시 할수는 없지만, 리더(후술)의 상태와 동일시하도록 만들 수 있다.

이 Raft algorithm 이 consensus algorithm 을 다룬 첫 논문은 아니다. Paxos, Viestamped Replication 같은 다른 알고리즘들도 있는데, Raft algorithm 이 나온 이유는 저 알고리즘들이 어려워서.. 이다. Raft 저자의 말을 살짝 빌리자면 내로라하는 학자들 중에서도 Paxos를 제대로 이해하고 있는 분들은 손에 꼽고, 그로 인해 실제로 구현해서 사용하기가 너무 어렵다. 이 때문에, Paxos-like 한 방식으로 consensus algorithm을 구현하게 되는데, 결국은 원래의 알고리즘의 목적에 맞지 않은 코드가 나와 error-prone 한 코드가 되고, 목적하고자 하는 바를 이루지 못하게 된다. 그래서 Raft 알고리즘 paper 에서 처음부터 강조하고 있는 것이 바로 understandability 이다.

이 이해하기 쉬운 합의 알고리즘을 만들기 위해 문제를 나누어 접근하고, 관리할 state 의 절대적인 개수를 줄였다.

Raft 는 Consensus Algorithm의 문제를 크게 4가지로 나누었다.

1. leader 선출
1. log 복제
1. saftey
1. membership changes


Raft Algorithm 에서 항상 보장하려는 것

1. Election Saftey : leader는 최대 1명이다.
1. Leader Append-Only : replica 되는 기준은 항상 leader 이다.
1. Log Matching : 같은 index 와 term 값을 가지고 있다면, 동일한 log[] 를 가지고 있다는 것을 의미한다.
2. Leader Completeness : log 가 어떤 term 에서 commit 이 됐다는 말은, 리더는 이미 그 log 를 포함하여 모든 log 를 가지고 있다는 말이다.
3. State Machine Safety : 만약 한 서버가 log entry 에서 어떤 index 의 log를 반영 하였을 때, 모든 서버는 같은 상태를 보장한다. (다른 로그로 반영을 한 서버는 없다.)

여기서 Paxos랑 ViewStamped replication 과의 차이점이 나타난다. Raft Algorithm 은 Strong leader 가 있다. 이 leader 는 client 로 부터 요청(command, 이후 log entry 로 래핑한다)을 받고, 다른 서버들에 replica 를 저장한다. 그리고, leader를 고르기 위한 election 단계가 있는데, 이때 candidate 노드들은 election timeout 값을 랜덤하게 재조정한다.

투표 과정을 보면, 과반 이상의 투표를 받아야 leader 가 될 수 있다. 과반 이상의 node 가 장애가 생겨 투표를 할 수 없을 경우, leader 를 선출할 수 없다. Raft algorithm 은 leader가 요청을 받아 모두에게 전파하는 구조이기에, 이럴 경우 절반 조금 안되는 node가 살아있어도 클러스터 관리를 할 수 없는 상태가 된다.

## Raft algorithm 요약
Raft paper 4p 에 있는 전체 요약 번역해봤다.

### State
##### Persistent State for all server (RPC 에 응답을 하기 전에, storage 에 저장하는 값들이다.)

* currentTerm : 몇번째 term 인지를 나타낸다. boot 때는 0이다. 여기서 term은 CPU 에서의 clock cycle 처럼 논리적인 턴이라고 생각할 수 있다.
* votedFor : currentTerm 번째 투표를 해야한다면, 그때 투표할 id 이다. 이때 id 는 candidate 의 id 를 의미한다.
* log[] : log entries 다. 각각의 entry 는 (term, command) 이다.

##### 서버들의 volatile state

* commitIndex : 로그 entry 에 마지막으로 커밋된 로그 id
* lastApplied: 로그가 커밋된 이후, 실제로 적용된 가장 마지막 로그 id
* 리더의 volatile state ( 선거 후에 다시 초기화 한다.)

* nextIndex[] : 어레이의 크기는 서버 수만큼 있고, 각각의 index 는 그 해당 서버에 전달해줄 다음 log id 이다. 기본값은 leader 의 last index + 1
* matchIndex[] : 각각의 서버에 복제해둔 로그 id 중 가장 큰 값

### AppendEntries RPC

leader -> follower 에게 hearbeat 를 보내거나, log 를 보내거나 하는 경우 사용한다.

##### Arguments

* Term : 리더의 term
* leaderId: follower 가 client 에게 redirect 할 수 있도록 해준다.
* prevLogIndex
* prevLogTerm
* entries[] : 저장할 log entries / heartbeat 할때는 비어있고, 보통 효율적으로 하기 위해서 1개 이상을 보낸다)
* leaderCommit

##### Result

* term : currentTerm
* success : follower 가 가지고 있는 prevLogIndex 랑 prevLogTerm 으로 검증을 하고, 이상이 없으면 true 를 반환한다.

##### Receiver 구현:
1. term < currentTerm 일 경우, false 를 반환
2. prevLogIndex 인 log가 없을 경우, false
3. 기존에 있는 entry가 새로운 것과 conflict 가 날 경우, 원래 있던 entry 와 이후의 것들을 삭제한다.
4. 이후, log entry에 없는 모든 log 를 집어넣는다.
5. leaderCommit > commitIndex 일 경우, commitIndex = min(leaderCommi,t index of last new entry) 로 설정한다.

### RequestVote RPC
leader가 실패하였거나 죽었을 때, 다른 서버로 연결이 끊어진다면 새로운 리더를 선출한다. 이때 사용하는 RPC 이다.

* term : candidate 의 term 이다
* candidateId : 투표를 요청한 candidate 의 id 이다
* lastLogIndex
* lastLogTerm

##### Results

* term : candidate 는 결국 leader 의 volatile state를 업데이트 해야하는데, 그때 사용하라고 자신의 값을 던져준다.
* voteGranted : 요청을 보낸 candidate 에게 투표를 할 경우, true 를 반환한다.

##### Receiver 구현
1. term < currentTerm 일 경우, false
2. votedFor (persistent state 의 것) 이 null 이거나 candidate Id 일 경우, 그리고 후보자의 log가 투표자의 것보다 더 최신일 경우, 투표를 한다.

### Rules for servers
##### All servers

* commitIndex > lastApplied : lastApplied 를 올리고, 그 index의 log 를 state machine 에 반영한다.
* RPC req/res 의 term T가 currentTerm 보다 클 경우, currentTerm = T 로 하고 follower 가 된다.

##### Followers

* candidate 와 leader 의 RPC 에 응답한다.
* AppendEntries RPC를 리더에게서 못받고, granting vote 또한 후보자에게서 못받은 상태로 선거 타임아웃이 된다면, 후보자가 된다.

##### Candidates

* 후보자로 바뀐다면, 선거를 시작한다.
* currentTerm 을 증가시킨다
* 스스로에게 한표 주고
* election timer 를 초기화 한 후
* RequestVote RPC 를 모든 서버에게 보낸다.
* 과반수 이상의 서버에서 투표를 받는다면, leader 가 된다.
* 새로운 리더에게서 AppendEntries RPC 를 받는다면 follower 로 바뀐다.
* 리더가 되지도 않았고, 새로운 리더한테서 일이 안오면 선거를 다시한다.

###### Leaders

* 당선되면, 빈 body 의 AppendEntries RPC 를 하트비트 용으로 보낸다. 선거가 다시 시작하는 것을 방지하기 위해 재전송 주기는 election timeout 보다 짧다.
* client 로부터 command 를 받으면, local 에 log 를 저장하고 state machine 에 apply 한 후에 response 한다.
* last log index > follower 의 nextIndex 라면, AppendEntries RPC 를 보낸다.
* 성공시 nextIndex 와 matchIndex 를 업데이트 한다
* log inconsistency 때문에 실패하면 nextIndex 를 줄여 재시도 한다.
* N > commitIndex 인 N 이 있고, 과반수 이상의 matchIndex 가 N보다 크거나 같으며 log[N].term 이 current Term 일 때, commitIndex = N 으로 설정한다. (그냥 적당히 로그를 전체 서버 절반 이상 replica 저장 완료했다고 인지하면 올린다)

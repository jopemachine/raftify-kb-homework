# Keywords in the Raft paper

Raft 논문에서 나온 키워드를 살펴보자.

## 정의

- Safety 안전성: 절대 _틀린_ 결과를 리턴하지 않는다는 성질.
  - Consensus algorithms ensure **safety** (never returning an incorrect result) under all non-Byzantine conditions.

- State Machine 상태 기계: 유한 상태 기계. 모든 시점에 유한한 개수의 상태 중 하나에 있을 수 있는 일종의 가상 기계.

  - Consensus algorithms typically arise in the context of **replicated state machines**.
  - Raft 는 복제된 상태 기계 간 합의를 위한 알고리즘이다.

- Raft 에서는 아래 성질이 보장된다.

  - Election Safety: 하나의 term 에 대해서는 하나의 리더만 존재할 수 있다.

  - State Machine Safety 상태 기계 안전성: Raft 에서 Safety 를 보장하는 핵심 성질.
    - 어떤 서버가 로그 엔트리를 자신에게 적용했다면, 다른 그 어떤 서버도 동일한 인덱스에 대해 다른 로그 엔트리를 적용하지 않음.

  - Log Matching: 만약 두 로그에 동일한 인덱스와 term 을 가진 엔트리가 있다면 그 로그는 해당 인덱스까지의 모든 항목에 대해 동일한 엔트리를 갖고 있다는 성질.
  - Leader Append-Only: 리더는 절대 로그 엔트리를 지우거나 덮어쓰지 않는다. 새 엔트리를 추가할 뿐이다.
  - Leader Completeness: 로그 엔트리가 어떤 term 에서 커밋 되었다면 그보다 높은 숫자의 term 을 가진 리더의 로그에도 존재한다. 즉, 앞선 리더가 커밋한 로그 엔트리라면 나중에 선출된 리더의 로그에도 그 엔트리가 존재해야 한다.

- Leader 리더: 클라이언트의 쓰기 요청을 받아들이는 노드. (적고보니 ?? 읽기 요청도 리더에서만 ?? 강한 일관성이니까 ??)

  - Term: 단조증가하는 정수값. 새 리더가 확정될 때마다 1씩 늘어난다.
    - 리더는 팔로워 각각의 term 을 관리한다.


  - Leader Election 리더 투표: 리더가 누가 될 지 합의하는 과정.

- Follower 팔로워: 리더가 아닌 나머지 노드.

- Heartbeat: 리더가 팔로워에게 하는 생존신고.

- Election Timeout: Election 을 발동시키는 타임아웃. 팔로워가 새 투표를 시작하기 전까지 기다리는 시간.

  - 이 시간이 지났는데 리더로부터 heartbeat를 받지 못했다면 리더 노드의 장애라고 판단하고 투표를 시작한다.

- RPC
  - AppendEntries: 리더 -> 팔로워. 리더의 heartbeat 혹은 데이터 동기화 요청 용도로 보내는 RPC.
    - Args: term, leaderId, prevLogIndex, prevLogTerm, entries[], leaderCommit
    - Return: term, success
  - InstallSnapshot: 리더 -> 팔로워. 스냅샷 적용을 위해 보내는 RPC. 길이가 길면 순서대로 보낸다.
    - Heartbeat 는 아니지만 이걸 받았을 때에도 election timer 가 리셋된다.
    - Args: term, leaderId, lastIncludedIndex, lastIncludedTerm, offset, data[], done
    - Return: term
  - RequestVote: 팔로워 -> 팔로워. 새 투표를 시작할 때 보내는 RPC.
    - Args: term, candidateId, lastLogIndex, lastLogTerm
    - Retun: term, voteGranted

- Membership Change: 클러스터 구성 노드 정보를 담고 있는 설정이 바뀌는 것. 즉, 클러스터에 새 노드가 추가되거나 기존 노드가 사라지는 것.

  - Stable Store: 클러스터 외의 다른 안전한 저장소에 멤버십 정보를 저장한다. 그렇지 않으면 클러스터 장애 발생 시 이 정보가 오염되거나 사라질 수도 있기 때문.

- Joint Consensus: 클러스터 멤버십 설정을 전환하는 중간에 old, new 멤버 명단을 모두 담고 있는 임시 설정.

  - 이게 없으면 disjoint majorities 가 발생할 수 있다.

- Log Compaction 로그 압축: 새로 들어온 노드에 그동안의 로그를 빠르게 적용시키기 위해 로그를 압축하는 것
  - Snapshot 스냅샷: Raft 에서는 간단한 방법인 스냅샷으로 로그 압축을 한다.

## About Raft

* Server State 서버 상태
  * 서버는 리더, 팔로워, 후보 3가지 상태 중 1가지를 가진다.
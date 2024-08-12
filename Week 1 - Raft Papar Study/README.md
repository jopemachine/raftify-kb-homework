## Raft Overview

Raft 합의 알고리즘은 분산 합의 구현체이다
Raft의 노드는 3가지의 상태를 가진다
- Follower
- Leader
- Candidate

**Leader Election Overview**
1. 모든 노드는 follower 상태로 시작하며, Leader로부터 아무것도 듣지 못한다면 candidate가 될 수 있다. 
2. Candidate는 다른 노드들에게 투표를 요청한다.
3. 노드들은 투표에 대해 응답한다.
4. 다수결(majority) 노드로부터 투표를 얻게 된다면, candidate는 leader가 된다.

**Log Replication Overview**
1. 각 변화는 노드 로그(log)의 entry에 추가된다.
2. Entry를 commit하려면 follower 노드들에게 복제를 해야한다.
3. Leader는 다수결(majority)의 노드들이 entry에 쓰기할 때까지 기다린다.
4. Leader 노드의 entry에 commit되고 Leader 노드의 상태는 바뀐다.
5. Leader는 follower들에게 entry가 commit됐다고 알린다.
6. Cluster는 시스템 상태에 대해 합의(consensus)를 이루게 되었다


Leader가 선출된 후, client의 요청이 시작된다. Leader는 client의 command를 새로운 entry log에 붙여(append), [[AppendEntries]] RPC를 entry를 복제(replicate)하기 위해 다른 서버들에게 요청한다.
Entry가 잘 복제되면, leader는 [[State Machine]]에 entry를 적용하고, client의 요청을 실행하여 결과값을 반환한다.
반대로 follower들이 문제가 생겼거나 실행이 느리면, leader는 Append Entries RPC 요청을 모든 follower들이 모든 log entries에 저장할 때까지 다시 시도한다.

Entry가 _committed_ 됐다면 leader는 log entry를 state machine에 적용하기 안전하다고 생각한다. Raft는 커밋된 entry들이 지속가능하고 모든 사용한 가능한 state machine이 실행할 수 있도록 보장한다.
Leader는 지속적으로 가장 높은 index를 추적하여 commit된 상태를 확인하고 나중에 보낼 Append Entries RPC에 포함한다.

Raft는 다른 서버의 log들의 높은 수준의 통일성을 유지한다. 시스템의 행동을 단순화하고 예측가능하게 만들어 안전성을 보장한다.
[[Log Matching]]
- 만약 다른 log에 있는 두 entry가 같은 index와 term을 가지고 있다면, 그들은 같은 command를 저장한다
- 만약 다른 log에 있는 두 entry가 같은 index와 term을 가지고 있다면, 선행된 entries의 log들은 동일하다

첫 번째 특성은 leader는 주어진 term과 index로 1개의 entry

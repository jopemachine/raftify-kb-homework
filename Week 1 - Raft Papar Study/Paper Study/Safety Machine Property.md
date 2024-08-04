Raft는 아래와 같은 성질들이 항상 보성립됨을 보장한다.

Election Safety: 주어진 term마다 하나의 리더만 선출된다.
[[Leader Append-Only]]
[[Log Matching]]
[[Leader Completeness]]
State Machine Safety: 어떤 서버가 로그 엔트리를 주어진 index에 상태 머신에 적용할 경우 다른 서버들은 해당 index에 다른 로그 엔트리를 적용할 수 없다. 해당 Property는 Raft의 안정성을 위한 Key Property이다.
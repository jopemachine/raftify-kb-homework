하나의 index에서 서로 다른 두 log entry가 state machine에 적용될 수 없다


[[Leader Completeness]]가 성립하면 State Machine Safety Property가 성립한다
- 어느 log entry L을 state machine에 적용한다고 하자. log entry L은 commit되었을 것이고, 그러면 해당 term의 leader log에 포함되어 있다.
- 시스템에 포함된 모든 노드 중 log entry L이 갖을 수 있는 최소한의 Term T에 대해서 Term T보다 높은 Term에 있는 Leader는 모두 log entry L을 포함한다.
- 따라서 이후에도 항상 같은 log entry L을 적용한다.
- 이는 하나의 index에서 항상 같은 log entry가 state machine에 적용됨을 의미한다.

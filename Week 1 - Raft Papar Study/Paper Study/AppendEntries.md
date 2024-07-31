AppendEntries는 leader의 log entry 복제와 heartbeat를 위해 사용된다

#### Arguments

| 이름           | 의미                                    |
| :----------- | :------------------------------------ |
| term         | leader의 term                          |
| leaderId     | follower redirect clients             |
| prevLogIndex | 이전에 log entry의 index 값                |
| prevLogTerm  | preLogIndex entry의 term 값             |
| entries[]    | 저장한 log entry array, heartbeat는 empty |
| leaderCommit | leader의 commit index                  |

#### Results

| 이름      | 의미                                                   |
| ------- | ---------------------------------------------------- |
| term    | 현재 term                                              |
| success | follower가 가고 있는 entry prevLogIndx와 prevLogTerm 매칭 여부 |

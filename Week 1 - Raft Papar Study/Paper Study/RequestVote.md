RequestVote는 candidate의 RPC 요청을 통해 시작한다.

#### Arguments

| 이름           | 의미                               |
| ------------ | -------------------------------- |
| term         | candidate의 term                  |
| candidateId  | candidate 요청 투표                  |
| lastLogIndex | candidate의 마지막 log entry index 값 |
| lastLogTerm  | candidate의 마지막 log entry term 값  |
|              |                                  |
#### Results

| 이름          | 의미                         |
| ----------- | -------------------------- |
| term        | 현재 term                    |
| voteGranted | candidate가 투표를 얻게 된다면 true |

--- 
###### 투표요구
`RequestVote` RPC가 이러한 제한을 구현한다. 이 RPC는 후보자 로그의 정보를 포함하며, 투표자는 자신의 로그가 후보자의 로그보다 더 최신일 때 투표를 부정하게 된다.

### RequestVote RPC

`RequestVote`는 투표를 위해 후보자에 의해 호출된다.

#### RequestVote 인자 (Arguments)

- `term`: 후보자의 [[term.md|term]] 값.
- `candidateId`: 투표를 요청하는 후보자의 `id`.
- `lastLogIndex`: 후보자의 마지막 로그 엔트리의 `index`.
- `lastLogTerm`: 후보자의 마지막 로그 엔트리의 `term` 값.

#### RequestVote 리턴 값 (Results)

- `term`: 후보자를 업데이트 하기 위한 `currentTerm` 값.
- `voteGranted`: 후보자가 표를 받았다면 _True_.

#### RequestVote 메시지 수신부 구현

1. `term` < `currentTerm` 인 경우 _False_를 리턴.
2. 만약 `votedFor`이 _null_이거나 `candidateId`이라면 후보자의 로그는 적어도 수신자의 로그와 업데이트된 상태이며, 투표한다.
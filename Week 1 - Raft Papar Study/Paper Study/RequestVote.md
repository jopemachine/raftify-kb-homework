# RequestVote

RequestVote RPC는 리더 선출 과정에서 후보가 팔로워들에게 투표를 요청하는 RPC이다.

## RequestVote의 동작 방식

RequestVote는 아래와 같은 필드를 포함하다.

- `term`: 후보의 term
- `candidateId`: 투표를 요청하는 후보의 ID
- `lastLogIndex`: 후보의 마지막 로그 항목 인덱스
- `lastLogTerm`: 후보의 마지막 로그 항목 회기

팔로워는 RequestVote RPC를 수신했을 시 아래와 같은 응답을 전송한다.

- 현재 term보다 작은 경우 `false`를 반환
- `voteFor`가 `null`이거나 후보의 ID와 같고, 후보의 로그가 수신자의 로그보다 최신이면 투표를 승인한다.

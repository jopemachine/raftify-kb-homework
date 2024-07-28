# AppendEntries RPC

AppendEntries RPC는 리더가 팔로워에게 로그 엔트리를 전송할 때 사용하는 RPC 종류 중 하나이다.

## AppendEntries RPC의 역할

- **Log Replication (로그 복사)**
  - 리더는 클라이언트로부터 새로운 로그 엔트리를 받으면, 이 엔트리를 자신의 로그에 추가하고, AppendEntries RPC를 이용하여 해당 로그 엔트리를 팔로워들에게 전송한다.
  - 팔로워는 AppendEntries RPC를 수신하면, 자신들의 로그에 해당
    엔트리를 추가한다.
  - 리더는 대부분의 팔로워들이 해당 엔트리를 설공적으로 추가하면, 이를 커밋하고, 클라이언트에게 결과를 응답한다.
- **하트비트 (Heartbeat)**
  - 리더는 일정 주기마다 AppendEntries RPC를 팔로워에게 전송하여 자신이 아직 리더임을 알리는 동시에 팔로워들이 후보자가 되는 것을 방지한다.(선거 타임아웃 방지)

## AppendEntries의 동작 방식

AppendEntries RPC는 다음과 같은 필드를 포함한다.

- `term`: 리더의 현재 임기
- `leaderId`: 리더 ID
- `prevLogIndex`: 새로운 엔트리를 추가하기 직전의 로그 인덱스
- `prevLogTerm`: `prevLogIndex`에 해당하는 로그 엔트리의 임기. 예를 틀어, 인덱스 3의 로그 엔트리가 임기 2에 차가되었다면 `prevLogTerm`은 2
- `entries`: 복제할 로그 엔트리 배열 (빈 값 가능)
- `leaderCommit`: 리더의 커밋된 로그 인덱스

<br>
AppendEntries RPC를 통해 팔로워가 확인하는 것은 아래와 같다.

- 임기 확인: 리더의 term이 팔로워의 term보다 작으면 AppendEntries를 거부
- 로그 일관성 확인: prevLogIndex와 prevLogTerm이 팔로워의 로그와 일치하지 않으면 AppendEntries를 거부
- 로그 추가: 위 사항들을 만족하였다면, 팔로워는 새로운 엔트리들을 자신의 로그에 추가
- 커밋 업데이트: leaderCommit이 팔ㄹ워의 커밋 인덱스보다 크면, leaderCommit에 따라 커밋 인덱스를 업데이트함

### LogEntry

<hr>

`RaftCore::raft_log`를 통해 로그 엔트리들을 접근한다. 커밋 전 엔트리들은 `Unstable::entries`에 존재하며 `on_ready`에서 이것들이 처리되어 커밋되게 된다.


- `entry_type`: 로그 유형을 나타냄. 일반적이거나 joint Concencus를 알아보기 위한 타입
- `term`: 로그에 기록된 term값
- `index`: 몇번 인덱스 로그였는지
- `data`
- `context`
- `sync_log`: 레거시 코드라고 한다.

위 데이터를 사용하여 Raft에서 로그항목을 구성하고 복제하는데 이용된다.
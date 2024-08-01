### RawNode

<hr>

`Raft`를 포함하여 `SoftState`, `HardState` 등을 포함하는 구조체이다. `RaftNode`에 해당 하는 `RaftNodeCore`에서는 이 `RawNode`를 포함 하고있다.

- `raft`: Raft 구조체
- `prev_ss`: `on_ready`에서 커밋 시기에 저장되는 SoftState
- `prev_hs` : `on_ready`에서 커밋 시기에 저장되는 HardState
- `max_number`
- `records`: `Ready`생성시 `record`가 저장된다.
- `commit_since_index`: 커밋된 로그항목 인덱스. commit_since_index+1로 offset을 설정하여 uncommitted_entries를 찾는다.
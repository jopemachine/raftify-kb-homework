--- 

Raft 노드를 업데이트 해야 할 필요가 있을 때 갱신되어야 할 데이터들을 한꺼번에 넘겨주는 자료구조

``` rust
/// Ready encapsulates the entries and messages that are ready to read,
/// be saved to stable storage, committed or sent to other peers.
#[derive(Default, Debug, PartialEq)]
pub struct Ready {
    number: u64,
    ss: Option<SoftState>,
    hs: Option<HardState>,
    read_states: Vec<ReadState>,
    entries: Vec<Entry>,
    snapshot: Snapshot,
    is_persisted_msg: bool,
    light: LightReady,
    must_sync: bool,
}
```
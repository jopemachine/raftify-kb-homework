# RawNode
Each Raft node has a higher-level instance called `RawNode` that includes the Raft module. The `RawNode` has a records field that represents `SoftState`, which is the **in-memory state**, `HardState`, which is the **state stored in persistent storage**, and Ready, which contains metadata that has not yet been saved.

```rust
pub struct RawNode<T: Storage> {
    pub raft: Raft<T>,
    prev_ss: SoftState,
    prev_hs: HardState,
    max_number: u64,
    records: VecDeque<ReadyRecord>,
    commit_since_index: u64,
}
```

When the ready() method is called, the metadata in Ready is saved to records, and **after processing all snapshots**, log entries, and other data that need to be stored, the RawNode::advance() function calls **RawNode::commit_ready() at the end to clear the snapshots** and entries in the Unstable buffer.

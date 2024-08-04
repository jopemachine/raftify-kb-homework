
```rust
pub struct Progress {
    pub matched: u64,           // Highest replicated log entry index
    pub next_idx: u64,          // Index of the next log entry to send
    pub state: ProgressState,   // Current state of the follower
    pub paused: bool,           // If true, sending logs is paused
    pub pending_snapshot: u64,  // Pending snapshot index, if any
    pub pending_request_snapshot: u64, // Index of requested snapshot, if any
    pub recent_active: bool,    // If true, follower was recently active
    pub ins: Inflights,         // Tracks inflight messages
    pub commit_group_id: u64,   // ID of the commit group
    pub committed_index: u64,   // Last committed index for this follower
}
```

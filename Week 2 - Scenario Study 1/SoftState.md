
```rust
pub enum StateRole {
    Follower,       // The node is a follower of the leader
    Candidate,      // The node could become a leader
    Leader,         // The node is a leader
    PreCandidate,   // The node could become a candidate, if prevote is enabled
}

pub struct SoftState {
    pub leader_id: u64,         // The potential leader of the cluster
    pub raft_state: StateRole,  // The soft role this node may take
}
```
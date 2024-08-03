--- 

```rust
pub type ProgressMap = HashMap<u64, Progress>;///nodeid&진행상

/// `ProgressTracker` contains several `Progress`es,
/// which could be `Leader`, `Follower` and `Learner`.
#[derive(Clone, Getters)]
pub struct ProgressTracker {
    progress: ProgressMap,

    /// The current configuration state of the cluster.
    #[get = "pub"]
    conf: Configuration, ///  클러스터 현재 상태 추적
    #[doc(hidden)]
    #[get = "pub"]
    votes: HashMap<u64, bool>, ///투표
    #[get = "pub(crate)"]
    max_inflight: usize,
    
    group_commit: bool,

```

```rust
/// The progress of catching up from a restart.
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Progress {
    /// How much state is matched.
    pub matched: u64,
    /// The next index to apply
    pub next_idx: u64,
    /// When in ProgressStateProbe, leader sends at most one replication message
    /// per heartbeat interval. It also probes actual progress of the follower.
    ///
    /// When in ProgressStateReplicate, leader optimistically increases next
    /// to the latest entry sent after sending replication message. This is
    /// an optimized state for fast replicating log entries to the follower.
    ///
    /// When in ProgressStateSnapshot, leader should have sent out snapshot
    /// before and stop sending any replication message.
    pub state: ProgressState, /// probe, replicate, snapshot
    /// Paused is used in ProgressStateProbe.
    /// When Paused is true, raft should pause sending replication message to this peer.
    pub paused: bool,
    /// This field is used in ProgressStateSnapshot.
    /// If there is a pending snapshot, the pendingSnapshot will be set to the
    /// index of the snapshot. If pendingSnapshot is set, the replication process of
    /// this Progress will be paused. raft will not resend snapshot until the pending one
    /// is reported to be failed.
    pub pending_snapshot: u64,
    /// This field is used in request snapshot.
    /// If there is a pending request snapshot, this will be set to the request
    /// index of the snapshot.
    pub pending_request_snapshot: u64,

    /// This is true if the progress is recently active. Receiving any messages
    /// from the corresponding follower indicates the progress is active.
    /// RecentActive can be reset to false after an election timeout.
    pub recent_active: bool,

    /// Inflights is a sliding window for the inflight messages.
    /// When inflights is full, no more message should be sent.
    /// When a leader sends out a message, the index of the last
    /// entry should be added to inflights. The index MUST be added
    /// into inflights in order.
    /// When a leader receives a reply, the previous inflights should
    /// be freed by calling inflights.freeTo.
    pub ins: Inflights,

    /// Only logs replicated to different group will be committed if any group is configured.
    pub commit_group_id: u64,

    /// Committed index in raft_log
    pub committed_index: u64,
}
```
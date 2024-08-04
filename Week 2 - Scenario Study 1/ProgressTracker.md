
```rust
/// `ProgressTracker` contains several `Progress`es,
/// which could be `Leader`, `Follower` and `Learner`.
#[derive(Clone, Getters)]
pub struct ProgressTracker {
    progress: ProgressMap,

    /// The current configuration state of the cluster.
    #[get = "pub"]
    conf: Configuration,        // Tracking a cluster's state
    #[doc(hidden)]
    #[get = "pub"]
    votes: HashMap<u64, bool>,  // Represent which node voted 
    #[get = "pub(crate)"]
    max_inflight: usize,

    group_commit: bool,
}
```
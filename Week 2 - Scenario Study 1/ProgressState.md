The state of the Progress

```
#[dervie(Debug, PartialEq, Eq, Clone, Copy, Default)]
pub enum ProgressState {
    #[default]
    Probe,      // Whether it's probing
    Replicate,  // Whether it's replicating
    Snapshot,   // Whether it's a snapshot
}
```

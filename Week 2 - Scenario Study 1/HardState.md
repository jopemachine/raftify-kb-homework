

```
message HardState{
    uint64 term = 1;    // The latest term that server got
    uint64 vote = 2;    // Voted node's id at existing term
    uint64 commit = 3;  // Latest CommitIndex(Involved SoftState in thesis)
}
```

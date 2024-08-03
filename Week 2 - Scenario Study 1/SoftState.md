--- 

```Rust
pub struct SoftState {
    pub leader_id: u64,
    pub raft_state: StateRole,
}
```

- 리더 아이디
	-  클러스터의 잠재적인 리더

- raft_state
	- 노드가 취할 수 있는 소프트 역할
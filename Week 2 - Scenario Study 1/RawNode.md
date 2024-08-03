--- 
 - [[RawNode|raft 모듈]]을 포함하는 더 하이레벨의 인스턴스
 - [[SoftState]] : 메모리에만 유지 되는 상태
 - [[HardState]] : 영속인 스토리지에 저장되는 상태
 - record : 아직 저장되지 않은 [[Ready]]의 메타데이터를 나타냄

Ready란 Raft 노드를 업데이트 해야 할 필요가 있을 때 갱신되어야 할 데이터들을 한꺼번에 넘겨주는 자료구조

``` rust
pub struct RawNode<T: Storage> {
	pub raft: Raft<T>,
	prev_ss: SoftState,
	prev_hs: HardState,
	max_number: u64,
	records: VecDeque<ReadyRecord>,
	commit_since_index: u64,
}
```

`ready()` 메서드가 호출될 때 `Ready`의 메타 데이터가 `records`에 저장되고, 저장되어야 하는 스냅샷, 로그 엔트리 등이 모두 처리된 후 함수의 마지막 부분인 `RawNode::advance()`에서 `RawNode::commit_ready()`를 호출하며 버퍼 `Unstable`의 스냅샷, 엔트리를 비웁니다.
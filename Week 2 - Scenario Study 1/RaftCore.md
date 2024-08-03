--- 
- raft 노드 [[Server State]]에 대한 데이터
- 서버의 persisted 상태는 raft log에서 다룸

```Rust
/// The core struct of raft consensus.
/// borrow check를 우회하기 위한 구조체

#[derive(Getters)]
pub struct RaftCore<T: Storage> {
    pub term: u64, /// 현재 선거기간
    pub vote: u64, // 이 node가 투표하는 peer
    pub id: u64, /// node id
    pub read_states: Vec<ReadState>,/// 현재 읽기 상태
    pub raft_log: RaftLog<T>, /// persist log
    pub max_inflight: usize, /// inflight에 있을 수 있는 최대 메세지 수
    pub max_msg_size: u64, /// 메세지 최대 길이
    /// follower가 snapshot을 담기 위한 인덱스
    pub pending_request_snapshot: u64,
    pub state: StateRole, /// node role
    promotable: bool, /// 리더로 격이 가능한지에 여부. true가 되면 cadinate
    pub leader_id: u64, /// leader id
    pub lead_transferee: Option<u64>, /// None이면 leader transfer id, Some이면 다른 절차
    pub pending_conf_index: u64, /// 한 번에 하나의 conf change만 보류 가능. 보류 중 log idx가 큰 값으로.
    pub read_only: ReadOnly, /// 요청 대기열
    pub election_elapsed: usize, /// leader&candinate일 때, election timeout이 발생한 이후 틱
    /// follower일 때, 마지막 electionTimeout에 도달하거나 현재 리더로부터 유효한 메시지를 받은 이후의 틱 수.
    heartbeat_elapsed: usize, ///마지막 heartbeatTimeout에 도달한 이후의 틱 수. 리더만.
    pub check_quorum: bool, /// 쿼럼 확인 여부
    pub pre_vote: bool, /// 클러스터 중단 전 투표 활성화 여부

    skip_bcast_commit: bool,
    batch_append: bool,

    heartbeat_timeout: usize,
    election_timeout: usize,
	
	/// election timeout 난수
    randomized_election_timeout: usize, 
    min_election_timeout: usize,
    max_election_timeout: usize,
    
    pub(crate) logger: slog::Logger,///로거
    pub priority: i64,/// 노드 선거 우선순위
    uncommitted_state: UncommittedState, ///커밋되지 않은 로그 항목
    pub(crate) max_committed_size_per_ready: u64,///커밋 항목당 최대 크
}
```

```rust
/// Raft log implementation
pub struct RaftLog<T: Storage> {
    pub store: T, ///  log storage 구현체
    pub unstable: Unstable, /// 스토리지 저장전에 거치는 버퍼 개념
    /// 저장하려는 스냅샷, 엔트리등이 있
	
/// 안정적인 저장소(quorum of nodes)에 저장된 가장 높은 로그 위치
/// applied <= committed 적용된 위치가 커밋된 위치보다 크지 않음을 보장
/// 재시작 후에 max_apply_unpersisted_log_limit가 0보다 크면 이 불변식이 깨질 수 있지만, 커밋된 로그가 적용된 로그를 따라잡으면 다시는 뒤쳐지지X
    pub committed: u64,
	
/// 안정적인 저장소에 영구히 저장된 가장 높은 로그 위치
/// 커밋되고 저장된 항목의 상한선을 제한
/// persisted < unstable.offset` 영구 저장된 위치가 불안정한 오프셋보다 작음을 보장
    pub persisted: u64,
	
///애플리케이션이 상태 머신에 적용하도록 지시받은 가장 높은 로그 위치
///`applied <= committed` 적용된 위치가 커밋된 위치보다 크지 않다는 것을 보장
///재시작 후에 `max_apply_unpersisted_log_limit`가 0보다 크면 이 불변식이 깨질 수 있지만, 커밋된 로그가 적용된 로그를 따라잡으면 다시는 뒤쳐지지 않습니다.
///max_apply_unpersisted_log_limit`가 0이면 `applied < persisted` 또한 보장됩니다 (이 값이 0보다 큰 값에서 0으로 변경되면, 적용된 로그가 영구 저장된 로그를 따라잡은 후에 보장됩니다).
    pub applied: u64,
	
// 적용된 로그랑 영구 저장된 로그 간의 간격
    pub max_apply_unpersisted_log_limit: u64,
}

```

```rust
/// The `unstable.entries[i]` has raft log position `i+unstable.offset`.
/// Note that `unstable.offset` may be less than the highest log
/// position in storage; this means that the next write to storage
/// might need to truncate the log before persisting unstable.entries.
#[derive(Debug)]
pub struct Unstable {
    /// The incoming unstable snapshot, if any.
    pub snapshot: Option<Snapshot>,

    /// All entries that have not yet been written to storage.
    pub entries: Vec<Entry>,

    /// The size of entries.
    pub entries_size: usize,

    /// The offset from the vector index.
    pub offset: u64,

    /// The tag to use when logging.
    pub logger: Logger,
}

```
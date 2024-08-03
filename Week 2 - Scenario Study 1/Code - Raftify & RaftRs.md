# [Raftify] Raft Loop

- Raftify-specific 용어: Raft 구현체에서 각 노드가 이 루프를 돌면서 자신의 상태를 업데이트 함. 즉, 하나의 raft 상태 머신 == Raft Loop 를 무한히 실행하는 프로세스.

## [Raftify] RaftNode

- Raftify 에서 RawNode 를 네트워크, 스토리지 계층과 통합해 좀 더 하이레벨에서 추상화 한 타입.
- `run()` 함수를 호출해 다음과 같이 동작함.
  1. 설정값으로 넘긴 `tick_interval` 로 tick timer 를 셋팅하고 앞으로 경과 시간을 측정하기 위한 `now`를 정의
  2. 무한히 루프를 돌면서 메세지를 폴링함. how to? `tokio::select!` 매크로를 사용함.
     1. 여러개의 비동기 동작을 _동시에_ 실행할 수 있는 `tokio::select!` 매크로 사용
     2.  `self.server_rcv.recv()` 가 fixed_tick_timer 의 타임아웃 이전에 완료되면 즉, 타임아웃 되기 전에 새 메세지를 받으면 해당 메세지를 처리함.
        1. how to? 메세지를 성공적으로 수신해서 `Ok(Some(msg))` 를 받았다면 메세지 핸들러를 호출함.
     3. 위 작업을 하는동안 경과된 시간(`elapsed`)을 체크
        - tick_timer 보다 더 많은 시간이 흘렀다면: `raw_node.tick()` 을 호출
        - tick_timer 보다 적은 시간이 흘렀다면: `tick_timer -= elapsed`
     4. `on_ready()` 함수 호출 - 만약 에러가 났다면 밖으로 전파함.

# Structures

## HardState

노드의 persisted 상태

- term: u64: 노드의 현재 term 값
- vote: u64: 현재 term 에 대해 리더 투표에서 표를 던진 노드의 아이디. 없는 경우 null.
- commit: u64: 커밋 된 엔트리 중 가장 높은 index 값

## SoftState

노드의 휘발성 상태. 메모리에만 저장됨. 

* `leader_id`:  u64: 현재 리더의 아이디
* `raft_state`: StateRole: 노드의 soft role

## RaftLog

노드가 저장하는 로그.

- `store`: T: 마지막 스냅샷 이후로 갖고 있는 모든 stable 엔트리를 이곳에 저장. Raftify와 같은 하이레벨 구현체에서 이 T 를 넣어줘야 한다.
- `unstable`: Unstable: 영속적인 스토리지에 기록되기 전 로그 엔트리와 스냅샷을 저장하는 버퍼.
- `committed`: u64: 커밋된 것으로 알려진 가장 마지막 엔트리의 위치 (? 이거 인덱스인가)
  - **applied <= committed**
- `persisted`: u64: Stable 저장소에 들어간 가장 마지막 엔트리의 위치
  - **persisted < unstable.offset**
- `applied`: u64: 상태 머신에 적용하라고 알려진 가장 마지막 엔트리의 위치
  - **applied <= committed**
- `max_apply_unpersisted_log_limit`: u64: persisted 와 applied 의 차이의 최대값

## Ready

'읽을 준비가 된' 항목과 메시지를 캡슐화. 이 데이터를 읽어서 영속적인 스토리지에 저장하거나, 커밋하거나, 다른 노드에게 전송함. 노드의 상태를 갱신할 때도 쓴다.

- 노드가 들고 다니는 속성을 가짐

  - `ss`: Option\<SoftState\> --> RawNode 에 있는거.

  - `hs`: Option\<HardState\> --> RawNode 에 있는거.

  - `read_states`: Vec\<ReadState\> --> RawNode.Raft.RaftCore 에 있는거.

- `entries`: Vec\<Entry\>
- `snapshot`: Snapshot
- `is_persisted_msg`: bool
- `light`: LightReady
- `must_sync`: bool

## Entry

- 상태 머신에 적용되어야 하는 로그 엔트리. 각 필드의 용도는 엔트리 타입에 따라 결정된다.

- Entry 는 proto buf 파일에 정의되어 있다. 즉, language-nutral 함.

- Rust 코드로 생성된 Entry 는 이렇게 생김.

  - ```rust
    #[derive(PartialEq,Clone,Default)]
    pub struct Entry {
        // message fields
        pub entry_type: EntryType,
        pub term: u64,
        pub index: u64,
        pub data: ::bytes::Bytes,
        pub context: ::bytes::Bytes,
        pub sync_log: bool,
        // special fields
        pub unknown_fields: ::protobuf::UnknownFields,
        pub cached_size: ::protobuf::CachedSize,
    }
    ```



## ProgressTracker

N개의 Progress(= Leader, Follower, 혹은 Learner) 를 관리함.

- `progress`: ProgressMap = HashMap<u64, Progress>
  - key: u64: 노드의 아이디.
  - value: Progress: 노드의 상태.

## Progress

- `matched`: u64: 로그에 대한 처리가 얼마나 이루어졌는지.
- `state`: 평소 정상적인 상황에서 이 값은 `ProgressState.Replicate`

## RawNode

- 각 raft 노드가 갖는 인스턴스.
  - SoftState, HardState: 각각 메모리, 영속적인 스토리지에 저장되는 상태값.
  - records:  Ready 의 메타 데이터
  - raft: 합의에 대한 데이터.
    - Progress Tracker, Message 큐, RaftCore 를 가짐.


# Enums

## MessageType

- AppendEntry 에 사용: MsgAppend, MsgAppendResponse
- RequestVote 에 사용: MsgRequestVote, MsgRequestVoteResponse
- 스냅샷에 사용: MsgSnapshot
- Heartbeat 전송에 사용: MsgHeartbeat, MsgHeartbeatReponse

# Functions

## advance()

- 노드는 advance 류의 함수를 호출해서 자신의 버퍼를 flush 함.
  - 즉, 특정 엔트리가 노드에 들어왔을 때
    1. Unstable 에 저장되고
    2. Ready 를 통해 엔트리를 전달
    3. on_ready 핸들러 호출
    4. advance 류 함수 호출하여 Unstable 을 flush, 영구적 스토리지에 저장

## step()

다른 노드로부터 받은 메세지를 처리하는 핵심 함수.

- 메세지의 term 과 자신의 term 을 비교하고
- 메세지 타입과 자신의 역할에 따라 메세지를 처리함.

## tick()

- Output: 처리해야 할 준비 작업이 있으면 `true`, 
- 내부적으로 쓰는 logical clock 을 1 틱 올린다. 노드의 상태에 따라 이렇게 처리한다.
  - Leader 일 때
    - 자신의 heartbeat_timeout 이 지나면 heartbeat 를 보냄. (MsgBeat)
  - 그 외 (Follower, PreCandidate, Candidate) 역할일 때
    - 자신의 election_timeout 이 지나면 `election_elapsed` 값을 0으로 초기화하고 MsgHup 을 발행 --> 해당 메세지를 `hup()` 에서 처리함.
      - `hup()` 에서는 새 선거를 시작해도 안전한지 체크한 다음, 캠페인을 시작함.

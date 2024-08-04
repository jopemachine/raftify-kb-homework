## voter 와 learner

- Raft 노드는 투표 참여 여부에 따라 voter 와 learner 로 나눌 수 있다.
- voter: 투표에 참여하는 노드
- learner: 투표에 참여하지 않는 노드

## ConfState

클러스터 멤버십을 관리하기 위한 구조체.

- `voters`: 투표에 참여하는 노드의 아이디 목록
- `learners`: 투표에 참여하지 않는 노드의 아이디 목록
  - 예) 새로 클러스터에 참여한 뒤로 계속해서 스냅샷 처리 단계인 노드.
- `learners_next`: learner가 되어야하는 노드의 목록. voters, learners 간 교집합은 없어야하기 때문에 voters 에 속했다가 learner 로 강등되어야 하는 노드의 경우 이곳에 잠시 머무르게 된다. 다른 learners 와 마찬가지로 투표에 참여하지 않는다.
- `voters_outgoing`: outgoing config (incoming config 로 대체될 config) 에만 속하는 노드 목록.
- `auto_leave`: old -> new 로의 전환을 Raft에서 한다면 true, 애플리케이션에서 한다면 false.

# eraftpb

- 네트워크 계층에 사용되는 타입은 eraftpb.proto 파일에 정의됨
- --> tonic 라이브러리에 의해 러스트 코드로 컴파일 되고
- --> eraftpb.rs 파일이 만들어짐.

```protobuf
message ConfState {
    repeated uint64 voters = 1;
```

```rust
impl ConfState {
    pub fn new() -> ConfState {
        ::std::default::Default::default()
    }

    // repeated uint64 voters = 1;


    pub fn get_voters(&self) -> &[u64] {
        &self.voters
    }
    pub fn clear_voters(&mut self) {
        self.voters.clear();
    }

    // Param is passed by value, moved
    pub fn set_voters(&mut self, v: ::std::vec::Vec<u64>) {
        self.voters = v;
    }

    // Mutable pointer to the field.
    pub fn mut_voters(&mut self) -> &mut ::std::vec::Vec<u64> {
        &mut self.voters
    }

    // Take field
    pub fn take_voters(&mut self) -> ::std::vec::Vec<u64> {
        ::std::mem::replace(&mut self.voters, ::std::vec::Vec::new())
    }
```

## Snapshot

로그 엔트리를 바탕으로 스냅샷을 만듦. 데이터와 메타데이터필드를 갖는다.

### SnapshotMetadata

스냅샷 메타데이터.

- `conf_state`: 스냅샷이 생성될 당시의 configuration
- `index`: 스냅샷 마지막 엔트리의 인덱스
- `term`: 스냅샷 마지막 엔트리의 term.

## MsgSnapshot

스냅샷 전송을 하기 위해 리더는 MsgSnapshot 타입의 메세지를 전송함.

- `step_leader()` 에서는 이 타입 메세지를 처리하지 않음.
- `step_candidate()` 에서는 이 타입 메세지를 받으면
  - `become_follower()` 를 호출해서 자기 자신을 팔로워 상태로 만들고
  - `handle_snapshot()` 을 호출해서 스냅샷 데이터를 처리한다.
- `step_follower()` 에서는 이 타입 메세지를 받으면
  - 일단 메세지를 받았기 때문에 `election_elapsed` 를 0으로 초기화하고 `leader_id` 를 메세지의 from 값으로 업데이트
  - `handle_snapshot()` 을 호출해서 스냅샷 데이터를 처리한다.

## propose_conf_change()

Config 변경을 _제안_ 함.

1. MsgPropose 타입 메세지를 만들고
2. EntryConfChange 혹은 EntryConfChangeV2 타입 엔트리를 만들어서 메세지에 넣음
3. 그 메세지로 `RawNode.step()` 호출 --> stemp 함수에서는 엔트리 타입이 위 타입 중 하나인걸 확인하고 적절히 핸들러 함수 호출.


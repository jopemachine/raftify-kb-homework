--- 
- 메시지 큐`msgs`를 메모리에 보유
- 이 큐를 통해 노드들간의 소통
- 우리가 만들 high-level 구현체의 네트워크 층에서 큐에 메시지를 넣는 역할을 함
고로, 메세지 큐는 통신의 엔드포인트로 간주. 이 큐에 있는 내용을 처리해 나가며 일관된 state를 유지함

- [[RaftCore]]
	- 노드 상태에 대한 데이터
- Progress 
	- 노드들간의 [[Log Entry|로그 엔트리]] 동기화를 위해 메타 데이터를 담음
	- 상황에 따라 [[ProgressTracker]]에서 업데이트

그러므로 이에 대해 코드로 표현하자면 노드는 다음과 같이 표현할 수 있다.

```Rust 
/// Raft 자체에 대한 구조체
/// 이 시스템이 가질 수 있는 상태&세부정보
pub struct Raft<T: Storage> {
    prs: ProgressTracker,

    /// The list of messages. 메세지 큐
    pub msgs: Vec<Message>,
    /// Internal raftCore. 노드 상태 정
    pub r: RaftCore<T>,
}
}```
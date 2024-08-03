--- 
- Raft 구현체-> 다른 Raft 노드들과 통신하며 일관된 상태를 유지하기 위해 무한 루프를 돌며 자신의 상태 머신을 업데이트 하는 반복적인 프로세스를 수행
- 이 글에선 이러한 루프를 _Raft loop_라고 부르겠습니다.
- on_ready 함수를 의미
- 복제되야하는 로그 시퀸스가 네트워크를 따라 복제
- 그 다음에 On_ready 함수를 거쳐 실제 스토리지에 저장.

``` rust
async fn on_ready(&mut self) -> Result<()> {
	if !self.raw_node.has_ready() {
		return Ok(());
		}
	
	let mut ready = self.raw_node.ready();
	
	if !ready.messages().is_empty() { 
		self.send_messages(ready.take_messages()).await;
	}
	
	if *ready.snapshot() != Snapshot::default() {
		slog::info!( 
			self.logger,
			"Restoring state machine and snapshot metadata..."
		);
		let snapshot = ready.snapshot();
		if !snapshot.get_data().is_empty() {
			self.fsm.restore(snapshot.get_data().to_vec()).await?;
		}
		let store = self.raw_node.mut_store(); 
		store.apply_snapshot(snapshot.clone())?;
	}
	
	self.handle_committed_entries(ready.take_committed_entries())
		.await?;
	
	if !ready.entries().is_empty() {
		let entries = &ready.entries()[..];
		let store = self.raw_node.mut_store();
		store.append(entries)?;
	}
	
	if let Some(hs) = ready.hs() {
		let store = self.raw_node.mut_store();
		store.set_hard_state(hs)?;
	}
	
	if !ready.persisted_messages().is_empty() {
		self.send_messages(ready.take_persisted_messages()).await;
	}
	
	let mut light_rd = self.raw_node.advance(ready);
	
	if let Some(commit) = light_rd.commit_index() {
		let store = self.raw_node.mut_store();
		store.set_hard_state_commit(commit)?;
	}
	
	if !light_rd.messages().is_empty() { 
		self.send_messages(light_rd.take_messages()).await; 
	}
	
	self.handle_committed_entries(light_rd.take_committed_entries())
		.await?;
	
	self.raw_node.advance_apply();
	
	Ok(())
	}
```

raftify는 아래와 같은 과정을 반복 처리합니다.

1. 클라이언트에서 요청 생성. (예를 들어 `RaftServiceClient.propose()`나 `RaftNode.propose()`를 호출)
2. gRPC를 통해 원격 Raft 노드의 `RaftServiceClient.propose()`가 호출됨.
3. `RaftServiceClient.propose()`가 채널을 통해 `Propose` 메시지를 `RaftNode.run()` 비동기 태스크로 넘김.
4. 메시지 큐를 폴링하던 `RaftNode.run()`은 `Propose` 메시지가 들어오면 `RawNode.propose()` 호출.
5. 상태 머신에 적용되어야 하는 변경 사항이 생기면 `Ready` 인스턴스가 생성되어 `on_ready()` 핸들러로 전달됨.
6. `on_ready()` 핸들러에서 커밋된 엔트리들을 처리한 후 클라이언트에 응답함.

💡 이 단락에서 Propose 메시지라고 임의로 칭한 것은 클러스터에 상태 변경을 제안하기 위한 목적으로 정의된 타입의 메시지입니다.
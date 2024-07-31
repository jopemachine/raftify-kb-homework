Raft 구현체들은 다른 Raft 노드들과 통신하며 일관된 상태를 유지하기 위해 무한 루프를 돌며 자신의 상태 머신을 업데이트 하는 반복적인 프로세스를 수행합니다. 이 글에선 이러한 루프를 _Raft loop_라고 부르겠습니다.


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
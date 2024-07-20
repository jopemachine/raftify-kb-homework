--- 

##### AppendEntries RPC Arguments
```
term: 현재 leader의 임기
leaderId : leader id
prevLogIndex : 바로 이전의 log index
prevLogTerm : 바로 이전의 log index에 기록된 임기(term)
entries[] : 저장할 로그 엔트리 (비어있으면 heartbeat용)
leaderCommit : 현재 leader의 commitIndex
```
1. Follower가 갖고 있는 현재 임기(`currentTerm`)보다 전달받은 `term`이 낮으면(`term`<`currentTerm`) -> **return false**
2. 전달 받은 `prevLogIndex`에 해당하는 엔트리가 없다면 -> **return false** 
	- (ex. leader의 index는 3인데 복사하려는 follower의 index가 1이어서 `prevLogIndex`인 2에 해당하는 entry가 없는 경우)
3. 이미 존재하는 엔트리가 새로운 엔트리와 충돌이 발생할 경우(같은index지만 다른term일 경우) -> **기존 엔트리 삭제**
4. **새로운 엔트리를 추가**
5. ==새로 추가된 엔트리의 index==와 ==leader의 commit Index== 중 작은 값을 follower의 commitIndex에 set
합의 형성 알고리즘.
네트워크, 스토리지 구현과 분리되어 있음.


Raft는 강한 일관성 모델.
대다수의 노드의 동의를 얻지 못하면 요청을 블록시키고 응답하지 않음.
raft는 tikv, etcd와 같은 강한 일관성 기반의 분산 키값 시스템에 사용.


### Raft 구현에 사용되는 RPC
기본적인 Raft 알고리즘에서 통신에는 두 종류의 RPC 필요. [[AppendEntries]] [[RequestVote]]
만약 RPC가 타임 아웃되면 재요청(Retry) 하게 되며 최적의 성능을 위해 RPC를 병렬로 요청.


### Log Replication

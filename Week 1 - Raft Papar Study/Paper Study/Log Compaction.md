시간이 지날수록 로그가 제한없이 계속 커지는 문제를 방지하기 위해 오래된 로그를 제거하는 방법

## 방법 1. Snapshot
Snapshot: 현재 시스템 상태 전체를 [[stable storage]]에 저장
- Snapshot이 저장되면, Snapshot에 포함된 log 전체를 삭제
- Chubby, Zookeeper에서 사용됨

Snapshot에는 어떤 정보가 포함되는가
- State machine state
- 메타데이터
	- last included index: 마지막 log entry의 index
	- last included term: 마지막 log entry의 term
	- last configuration: 마지막 log entry에서의 configuration

Snapshot의 일관성은 어떻게 보장하는가
 - [[InstallSnapshot]] 을 보내 Follower들이 snapshot을 복사하고, snapshot을 일치시키도록 보장
 - 만약 Leader만 snapshot을 만든다면 성능과 구현의 복잡성에 대한 문제가 발생한다

언제 Snapshot을 생성/저장하는가
- 로그 크기가 특정 바이트에 도달할 시
- (그 외 다른 방법 사용 가능)

Snapshot을 저장할 때 발생하는 딜레이는 어떻게 해결하는가
- Copy-On-Write 전략 사용
	- 복사본이 수정되었을 때만 새로운 복사본을 만드는 기법


## 방법 2. Incremental Approach
e.g. Log Cleaning, Log-Structured merge tree
전체 데이터가 아닌 일부데이터에 대해 동작

장점
- Compaction 부하가 더 균등하게 분산됨
단점
- 복잡한 추가 로직이 필요
- 적용을 위해 Raft 알고리즘을 변경해야할 수도 있음 (Log Cleaning)

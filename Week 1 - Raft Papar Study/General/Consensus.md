노드들 사이의 공유 메모리

**상태를 변화시키는 메커니즘**
여러 노드가 하나의 일관된 상태에 도달하기 위해 사용하는 프로토콜

분산 네트워크 환경에서는 Crash Fault Tolerant (CFT) 컨센서스와 Byzantine Fault Tolerant (BFT) 컨센서스로 나뉜다.

Raft 는 CFT 컨센서스로, 노드의 단순 고장 상황을 가정하여 분할 내성을 갖춤. [[Two Generals' Problem]] 상황을 가정.

BFT는 BFT 컨센서스로, 더 넓은 범위의 악의적인 행동과 관련된 문제 [[Byzantine faults]] 를 가정하여 분할 내성을 갖춤.
분할된 상황에서 [[Availability]]를 중시하는 POW, POS 등의 Chain Base 합의 알고리즘과
[[Consistency]]를 중시하는 텐더민트 등의 BFT Style 합의 알고리즘이 있음.


[[FLP Theorem]] 에 따라 async를 가정한 컨센서스는 안전하지 않고,

synchronous를 가정한 POW, POS 등의 합의 알고리즘은 네트워크의 Branch 상황을 수용하고, 체인 선택 규칙을 통해 일관성을 보장한다. (Eventual Consistency)

텐더민트, Raft 등의 알고리즘은 partially synchronous를 가정하여 [[Timeout]] 등을 통해 일관성을 보장한다. (Strong Consistency)
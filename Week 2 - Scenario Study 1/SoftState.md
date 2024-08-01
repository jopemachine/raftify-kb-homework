### SoftState

<hr>

프로세스가 종료되어 데이터가 사라져도 상관 없는 데이터 상태

다음과 같은 데이터가 있다.

`leader_id`: 현재 노드가 알고있는 현재 클러스터의 리더 id

`raft_state`: `StateRole`을 갖는다. 즉, 후보자, 팔로워, 리더 중 하나 (`PreCandidate`: 곧 후보자가 될 예정 상태 포함 총 4개)
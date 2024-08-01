### ProgressTracker

<hr>

클러스터 상태를 추적하고 관리한다. 

- `ProgressMap`: 클러스터 노드들 정보를 대표함. 클러스터 내 `Progress`들을 담고 있다.
- `Configuration`: 클러스터 상태를 대표함
- `votes`: 누가 누구에게 투표했는지에 대한 정보
- `max_inflight`
- `group_commit`

Progress와 Configuration을 포함한 구조체이기 때문에 이 구조체를 통해 하위 구조체에 접근하고 관리하는데 사용한다.
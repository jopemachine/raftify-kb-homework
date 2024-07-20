--- 
###### Raft는 아래와 같은 성질들이 항상 성립됨을 보장한다.
- Election Safety
	- 주어진 term마다 하나의 leader만 선출된다.
- Leader Append-Only
	- 리더는 절대 자신의 로그를 덮어쓰거나 삭제하지 않는다.
	- 오직 새로운 엔트리를 추가하기만 한다.
- [[Log Matching|Log Matching]]
	- 두 로그가 동일한 index와 term 값을 가진 엔트리를 포함한다면
	- 두 로그의 엔트리들은 주어진 그 엔트리의 index까지 모두 동일
- [[Leader Completeness.md|Leader Completeness]]
	- 만약 한 로그 엔트리가 어떤 term 값에 커밋된다면
	- 해당 엔트리는 더 높은 term 값들을 갖는 리더들의 로그에 존재
- **State Machine Safety** 
	- 어떤 서버가 로그 엔트리를 주어진 index에 상태 머신에 적용할 경우
	- 다른 서버들은 해당 index에 다른 로그 엔트리를 적용할 수 없다
	- 해당 Property는 Raft의 안정성을 위한 Key Property이다.

###### `Leader Completeness`를 통해 `State Machine Safety Property` 증명

`State Machine Safety Property`에 따르면 주어진 인덱스에서 서버가 로그 엔트리를 상태 머신에 적용했다면 다른 서버는 그 인덱스에 대해 다른 로그 엔트리를 절대 적용하지 않을 것이다.

서버가 로그 엔트리를 상태 머신에 적용할 때 그 로그는 해당 엔트리를 포함해 리더의 로그와 동일해야 합니다. 이제 주어진 로그 인덱스를 적용하는 서버가 있는 가장 낮은 term 값을 고려해보자. `Log Completeness` 속성은 모든 더 높은 임기의 리더들이 동일한 로그 엔트리를 적용할 것임을 보장한다. 따라서 나중 임기에서 로그를 적용하는 서버들은 같은 로그를 적용할 것이다. 따라서 `State Machine Safety Property`가 유지된다

마지막으로 Raft는 서버가 로그 인덱스 순서대로 엔트리를 적용하도록 요구한다. 이것은 `State Machine Safety Property`과 결합되어, 모든 서버가 동일한 로그 엔트리 집합을 자신의 상태 기계에 적용하게 될 것임을 의미한다.
### HardState

<hr>

영구적으로 저장해서 관리해야 하는 데이터 상태이다. `persisted` 속성을 갖는 값들로 `HardState`구조체에 담긴다.

논문 내용과 조금 다르게 다음과 같은 값들이 포함된다.

`term`: 현재 노드 기준 가장 최신의 `term`값

`vote`: 투표한 후보자의 id

`commit`: 로그엔트리중 커밋된 인덱스
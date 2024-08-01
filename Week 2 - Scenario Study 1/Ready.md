### Ready

<hr>

노드가 처리해야 할 모든 작업을 묶어서 하나의 구조체로 나타 냄. `Ready`에는 SoftState, HardState 같은 상태 데이터, 스냅샷, 엔트리들(unstable)이 포함되며 추가로 `LightReady`라는 구조체를 포함한다. 여기에는 커밋된 엔트리(인덱스와 같은 메타 데이터는 포함되지 않음), 메세지들(Rafg::msgs)을 갖는다. 

`LightReady`는 gen함수를 통해 생성되며, 이 과정에서 uncommitted인덱스를 업데이트한다는데 이부분에 대해서는 잘 모르겠다. 

`Ready`를 만들고 나면 메타 데이터를 담은 `ReadyRecord`를 만들어 추가해 준다. 이 레코드는 advance함수 내부에서 커밋하는 과정에서 사용된다.

`on_ready` 함수에서 이 `Ready`를 처리하는데, `Ready`내 데이터를 저장하고 이후 advance 함수 및 상태 기계에 반영하는 과정을 거친다. 

- 스냅샷 저장: 스냅샷을 store에 추가한다
- 로그 엔트리 저장: Ready가 갖는 로그 엔트리들은 unstable상태 데이터이기 때문에 store에 추가한다.
- 커밋된 로그 엔트리를 반영(?): `handle_committed_entries`이라는 함수를 호출하는데 정확하게 뭐하는 건지 모르겠다.
- HardState 저장
- 다른 노드에게 메세지 전달

이후 `advance`를 통해서 위에 저장된 데이터를 바탕으로 commit된다. 

advance함수 호출 이후 `LightReady`를 반환하는데 이 내용을 바탕으로 정보를 업데이트한다(handle_committed_entries)
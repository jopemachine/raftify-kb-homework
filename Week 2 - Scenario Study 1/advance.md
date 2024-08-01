### advance

<hr>

`Ready`를 처리하고나서 상태 기계에 반영하는 단계로 `commit_ready`에서 `Ready`의 데이터(ss,hs,엔트리 등)를 커밋하고, unstable 버퍼 데이터들을 비운다. `on_persist_ready`에서 `ReadyRecord`들을 순서대로 처리고, 커밋 인덱스를 업데이트 하는 과정을 거치며 커밋한다.

`advance`함수는 위 과정을 거친 후 `LightReady`를 반환하는데 추가된 커밋된 데이터 및 메세지 처리 등 다음 상태로 나아갔기 때문에 새로운 `LightReady`를 만든다.  

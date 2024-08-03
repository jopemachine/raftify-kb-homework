--- 

###### 리더는 `InstallSnapshot`이라는 새로운 RPC를 통해 뒤쳐진 팔로워들에게 스냅샷을 보낸다.

### InstallSnapshot RPC

`InstallSnapshot`은 스냅샷 청크들을 팔로워에게 보내기 위해 리더에 의해 호출된다.

#### InstallSnapshot 인자 (Arguments)

- `term`: 리더의 `term` 값.
- `leaderId`: 리더에게 요청을 리다이렉트 할 수 있도록 전달해주어야 하는 리더의 Id.
- `lastIncludedIndex`: 해당 `index`까지 스냅샷으로 생성.
- `lastIncludedTerm`: `lastIncludedIndex`에 해당하는 엔트리의 `term` 값.
- `offset`: 스냅샷 파일 청크가 위치하는 바이트 오프셋.
- `data`: 오프셋으로부터 시작하는 스냅샷 청크의 raw bytes.
- `done`: 이 청크가 마지막 청크인 경우 _True_.

#### InstallSnapshot 리턴 값 (Results)

- `term`: 리더를 업데이트 하기 위한 `currentTerm` 값.

#### InstallSnapshot 메시지 수신부 구현

1. `term` < `currentTerm`인 경우 즉시 응답.
2. 만약 첫 번째 청크라면 (오프셋이 0) 새 스냅샷을 생성.
3. 주어진 오프셋에서 스냅샷으로 데이터를 씀.
4. 응답하고, 만약 `done`이 _False_인 경우 더 많은 데이터 청크를 기다림.
5. 스냅샷을 저장. 기존에 존재하는 `index`가 작은 부분들은 모두 삭제.
6. 기존 로그 엔트리의 `index` 및 `term`이 스냅샷의 `index` 및 `term`과 동일하다면 로그에 마지막으로 포함된 엔트리와 해당 엔트리 뒤의 로그 엔트리들을 유지하고 응답.
7. 전체 로그를 삭제.
8. 스냅샷을 사용해 상태 머신을 리셋. (그리고 스냅샷의 클러스터 설정을 로드)
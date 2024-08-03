# 새 로그 엔트리 추가

## 리더

- 합의에 대한 변경사항을 만들기 위해 호출해야 하는 함수: `RawNode.propose()`

  - 예) Ratify 웹 서버 API 의 `GET /put/{id}/{value}` 에서는 이렇게 호출

    ```python
    raft_node = raft.get_raft_node()
    await raft_node.propose(message.encode())
    ```

### `RaftNode.propose()`

1. 메세지 생성: `MsgPropose` 타입, 자신의 아이디, 엔트리, context 등을 담음.

2. RawNode.raft.step(m) 호출

   - `step` 은 다른 노드로부터 메세지를 받은 경우 항상 호출되어야 함. MsgPropose 외에 다른 타입 메세지도 모두 핸들링하는 함수.

   - MsgPropose 타입에 대해서

     1. 엔트리가 비어있거나, 현재 config 에 존재하지 않는 노드로부터 받은 메세지가 아닌지 등등 예외 상황 확인

     2. 중간에 이것저것 좀 더 하다가 (나중에 다시 보기)

     3. `self.append_entry(m.mut_entries())` 호출

        - 리더에 의해서만 호출되는 함수.

        1. 현재 self.raft_log 의 마지막 인덱스를 가져오고

        2. 그걸 기준으로 각 엔트리의 index 를 직접 설정, term 도 셋팅한 다음에 (! 리더니까 직접 설정해야 함.)

        3. self.raft_log 에 추가함: `RaftLog.append(es)` 를 호출

           - Unstable 버퍼에 엔트리를 추가하고, 마지막 인덱스를 리턴하는 함수.

           1. 엔트리가 비어있으면 바로 현재 RaftLog 의 마지막 인덱스를 리턴
           2. 자신이 이미 커밋한 엔트리의 인덱스보다 작은 인덱스를 가진 엔트리가 들어오기 시작했다면, 에러 발생
           3. `RaftLog.unstable.truncate_and_append(ents)` 호출
              - 이것저것 한 다음 Unstable.entries 에 `ents` 를 추가
           4. 현재 RaftLog의 마지막 인덱스 리턴

     4. `self.bcast_append()` 호출

        - ProgressStatus 를 확인하고 자신을 제외한 나머지 non-up-to-date 노드에게 RPC 전송

        - `Raft.RaftCore.send_append()` 호출

          - `RaftCore.maybe_send_append()` 호출

            - 각 노드의 Progress 에 따라서 메세지를 처리.

            1. *pr.next_idx - to* 범위의 로그 엔트리를 읽어옴
            2. `RaftCore.prepare_send_entries()` 호출
               1. 받은 메세지 데이터를 바탕으로 MsgAppend 타입 새 메세지를 생성
               2. 엔트리가 비어있지 않다면 `Progress.update_state()` 호출
                  1. 자신의 ProgressState 상태가 Replicate 상태라면 (스냅샷 처리 중도 아니고, Probe 도 아닌 그냥 잘 있다! 상태.), 자신의 next_index 를 엔트리 개수 `n` + `1` 로 업데이트하고 inflights 에도 반영함.

            3. `RaftCore.send(m, msgs)` 호출
               1. 위에서 새롭게 생성한 `MsgAppend` 타입 메세지를 메세지 큐에 push. --> 팔로워가 이걸 받는다.

## 팔로워

- MsgAppend 를 팔로워가 어떻게 처리하는지 보려면 step_follower() 함수를 확인하면 된다.

### Raft.step_follower()

1. 자신의 election_elapsed 값을 0으로 초기화하고 (새 메세지 받았으니까!)
2. 리더 아이디를 받은 메세지에 있는 `from` 값으로 업데이트
3. `Raft.handle_append_entries(&m)` 호출
   1. 스냅샷 처리를 해야 할게 남아있지 않은지 확인하고
   2. 받은 메세지의 엔트리 인덱스가 자신이 이미 커밋한 인덱스보다 작지 않은지 확인하고
   3. MsgAppendResponse 타입의 새 메세지를 준비.
   4. `Raft.maybe_append()` 호출
      1. 메세지의 logTerm 과 엔트리의 term 값이 같은지 확인
      2. `find_conflict()` 로 충돌이 있는지 확인
      3. 아무 이상 없으면 `Raft.append()` 호출
   5. 호출 결과 이상 없으면 엔트리를 자기 자신한테 커밋하고
   6. 위에서 준비한 MsgAppendResponse 타입의 새 메세지를 send.
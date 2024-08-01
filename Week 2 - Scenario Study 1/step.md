### step

<hr>

`step`함수는 `propose`에서 받은 메세지를 처리하는 핵심 로직이 들어가있다. 메세지의 유효성을 검증하고 자신 노드의 상태에 따라 알맞은 로직을 수행한다.

`m`은 메세지를 의미하고 다음과 같은 검증을 시도한다.

- m.term==0 

  로컬 메세지로 특수한 경우에 사용

- m.term < self.term

  메세지의 term이 더 작은 경우 유효하지 않은 메세지이다. 일반적으로는 그냥 무시한다.

  선거 관련 메세지에 경우 특별히 `reject`플레그를 담아 응답하게 된다.  그 이유는 그냥 무시하면 나를 제외한 다른 노드들의 합의로 클러스터가 교착상태에 빠질 수 있다. 

- m.term > self.term

  가장 일반적인 상황으로 메세지의 term값이 높아 유효하다고 판단하고 적절한 로직을 수행하는 경우이다.

  `MsgAppend`,`MsgHeartbeat`,`MsgSnapshot`에 대해 `become_follower`로 팔로워로 전환되고, 다음 `step`에 진입한다.

위 조건에 알맞으면서 투표관련 메세지가 아닌경우 자신 상태에 따라 다음 `step`에 진입한다.



#### 리더 `step_leader()`

[propose](./propose)항목에서 `MsgPropose`를 전달한다고 했다. match에 MessageType::MsgPropose부분을 들여다 보면 먼저 메세지 자격증명(빈 메세지 확인,클러스터 구성원 확인 등)을 수행한 후 유효한 메세지에 경우 `append_entry`를 호출하여 `truncate_and_append`라는 함수에서 unstable에 엔트리들을 추가한다. 

리더가 팔로워의 로그 엔트리 불일치를 해결하는 방법으로 로그 엔트리를 덮어 쓴다고 하는데 `append` 함수에서 이를 처리하는것 같다.

자신에게 해당 엔트리를 추가 후 `bcast_append`를 호출해 클러스터 내 다른 노드들에게 메세지 복제를 시도한다. `maybe`류 함수를 내부에서 호출하는데 이는 전달이 완료되면 true, 실패하면 false를 리턴해서 정상적으로 메세지가 전달되었는지를 판단한다. 

기존 메세지 타입은 `MsgPropose`이였다. 하지만 `prepare_send_entries`함수 내부에서는 메세지 타입을 `MsgAppend`로 변경하여 `msgs`에 넣는다. 이러한 메세지들은 `on_ready`에서 처리하는 과정에서 메세지를 보내게 된다.



#### 팔로워`step_follower()`

리더가 보낸 `MsgAppend`를 받는 팔로워는 `handle_append_entries`를 호출하여 전달받은 메세지를 처리하고 `MsgAppendResponse`메세지를 다시 보내 확인 메세지를 보낸다.

팔로워 또한 `maybe_append`함수를 호출하고 내부에서 `append`함수를 호출해 unstable에 전달받은 엔트리들을 저장하고 `on_ready`에서 이를 처리한다.



#### 후보자`step_candidate()`

타임아웃이 일어나 리더로 교체될 필요가 있을 경우 `campaign`에서 후보자로 변하며 자신은 자신에게 투표를하고(`poll`호출), `MsgRequestVoteResponse`에 대한 처리를 하여  투표결과를 기다리며 과반수 이상을 만족하는 투표결과를 얻으면 리더로 선출되게 된다.

특수한 경우(조인트 쿼럼)의 경우 두 경우 모두 만족해야 `Won`플래그를 얻을 수 있다.

```
투표 집계 결과인 tally_votes에서 vote_result 호출
(VoteResult::Won, VoteResult::Won) => VoteResult::Won,
```


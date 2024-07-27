Leader의 log entries를 follower node에 복제하기 위한 요청 메세지
정상적으로 통신이 완료될 경우, Leader와 Follower 들 간 log는 일치하게 된다.

Client가 시스템에 특정 명령을 요청하면 Leader 에는 해당 요청이 log에 저장된다.
데이터 일관성을 위해 Leader는 업데이트 된 log를 시스템 내 Follower들과 일치시키기 위한 작업을 진행한다.

## 통신 형식

![[Pasted image 20240725181146.png]]
- Caller: Leader
- Callee: Follower
- Request Message Format
	- Term: Leader의 term
	- Leader ID: Leader의 ID
	- Prev log index: Leader가 갖고 있는 마지막 log entry의 index
	- Prev log term: Leader가 갖고 있는 log의 마지막 entry에 있는 term
	- Leader commit: Leader log 중 가장 마지막에 commit된 entry의 index
	- Entries: Leader가 갖고 있는 log entry
- Response Message Format
	- Term: 요청받은 term (Request Message Format과 동일)
	- Success: 응답하는 Follower의 log에 prev log index가 있고, 그 index에 해당하는 term이 동일한 경우 True 전달

## 동작
Leader가 자신의 log를 Follower에게 복제시킨다.

### Leader
1. Leader와 Follower 의 log가 일치하는 지점을 찾는다

Leader는 nextIndex에 각 Follower 별로 log가 일치하는 지점 다음 index값을 저장한다
- Leader는 nextIndex 값을 AppendEntries /요청의 prev Log Index에 넣어 follower에게 전달한다
- 만약 Follower로부터 (success = False) 값을 응답받으면 nextIndex값을 1 감소시키고, 감소시킨 nextIndex로 AppendEntries 요청을 다시보낸다. 이 과정을 (success = True)를 응답받을 때까지 반복한다

### Follower
AppendEntries 요청메시지를 받은 후 Follower는 다음과 같은 작업을 진행함

1. Success 값 설정
다음 상황에서 Success =  False로 설정
- 자신의 Term > 요청 메세지의 Term
- 자신의 log에 요청 메세지의 prev log index와 prev log term에 해당하는 entry가 존재하지 않음
그 외에는 Success = True로 설정

2. Conflict 해결
Conflict: 요청 메세지에 있는 log entry와 자신의 log entry를 비교해서, 같은 index에 다른 term이 있는 경우
Conflict이 발생한 경우, 자신의 entry를 요청메세지의 entry로 교체한다

3. log entry 추가
요청 메세지에 자신의 log에 없는 entry를 자신의 log에 추가

4. Commit Index 갱신
Leader Commit Index가 자신의 Commit Index보다 크다면 자신의 Commit Index 업데이트
- 가장 최신 Entry의 Index와 Leader Commit 중 더 작은 값으로 업데이트


Leader혹은 Follower에 장애가 발생하면 데이터 일관성이 깨질 수 있음
cf) 데이터 일관성이 깨진다
- Follower가 현재 Leader에게 있는 entry가 없다
- Follower가 현재 Leader에게 없는 entry가 있다

## 결과
Leader 와 Follower의 log entry값이 일치한다
- AppendEntries 요청이 성공한 시점 (success=True) 에서
- Follower는 conflict난 log entry를 모두 제거하고 (Follower - 2.Conflict 해결)
- Conflict 이후 Leader의 log entry는 follower의 log에 복사한다 (Follower - 3. log entry 추가)

## 기타
Rejected AppendEntires RPC의 수를 최소화하는 것이 좋지만, Failure 자체가 드물기 때문에 최적화 우선순위 가 낮다

## Question
Follower가 검사를 하지 않고, leader log를 통째로 복사하지 않을까?
- 검사 비용 vs 복사 비용 ?

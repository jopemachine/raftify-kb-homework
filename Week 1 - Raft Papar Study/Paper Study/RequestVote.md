[[Leader Election]] 에서 Candidate 이 리더로 선출되기 위해 시스템 내 다른 참여자에게 보내는 요청 메세지
과반수 이상의 응답메세지에서 voteGranted=True가 되면 해당 Candidate는 Leader가 된다

## 통신 형식

Caller
- Candidate

Callee
- 그 외 모든 Node

요청 메세지
- Term: Candidate의 Term
- Candidate ID: Candidate의 ID
- Last Log Index: Candidate의 마지막 log entry index
- Last Log Term: Candidate의 마지막 log entry term

응답 메세지
- Term: 요청받은 Term
- VoteGranted: Candidate에게 투표할지 여부
1. If votedFor is null or candidateId, and candidate’s log is at
    
    least as up-to-date as receiver’s log, grant vote
## 동작
### Receiver
다음 셋중 하나라도 만족하면 VoteGranted = False 로 설정
- Current Term < 요청 메세지의 Term
- votedFor != null ^ votedFor != 요청메세지의 Candidate ID
- 요청 메세지의 last log index와 last log entry가 up-to-date 인 경우
	- up-to-date
		- 두 log의 마지막 entry에서 term이 다른 경우, 더 큰 term을 가진 log가 up-to-date
		- 두 log의 마지막 entry에서 term이 같은 경우, 더 큰 index를 가진 log가 up-to-date
	- Leader가 항상 모든 committed log entry를 갖음을 보장하기 위해 up-to-date 여부 확인

그 외 경우, voteGranted = True로 설정

### Candidate
- 과반수 이상의 (voteGranted = True)를 받으면 Leader로 변경
- 그렇지 않으면 랜덤한 일정기간동안 대기
Follower가 Leader의 snapshot 파일을 복사


Calller
- Leader

Callee
- Follower

요청 메세지
- Term: Leader의 Term
- Leader ID: Leader의 ID
- last included index: 
- last included term
- offset: snapshot file에서 chunk 위치 (byte offset)
- data: offset부터 시작한 snapshot chunk data
- done: 마지막 chunk인지 여부

응답 메세지
- Term: 요청 메세지로부터 받은 Term

동작
1. current Term < 요청메세지의 Term이면 바로 반환
2. 새로운 snapshot 파일 생성
	- Done이 True일 떄까지 더 많은 data chunk요청
3. snapshot 파일 저장
4. 저장된 snapshot보다 더 작은 index의 이전 snapshot 제거
5. 현재 log가 snapshot의 last included entry 와 일치하는 entry를 포함하고 있는지 확인
	1. 일치한다면 바로 반환
	2. 일치하지 않는다면
		1. 로그제거
		2. snapshot을 사용해 state machine 초기화

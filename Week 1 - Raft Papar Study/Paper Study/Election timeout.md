Follower node는 Election timeout동안 [[heartbeat]]를 받지 못하면 Candidate node로 역할을 바꾸어 [[Leader Election]] 과정을 시작함

만약 모든 node의 Election timeout 값이 같다면, Leader를 선정할 수 없음.
- 모든 node가 동시 Candidate node로 역할을 변경
- 동시에 자기자신을 투표하고 [[RequestVote]] 메시지를 같은 시스템 내 node에게 전달
- 모두 자기자신을 투표했기 때문에 다른사람에게 투표하지 않음
- 모든 Candidate node가 과반수 투표를 받지 못해 leader 선정을 하지 못함

선거 과정을 무한히 반복하지 않기 위해 election timeout은 다음과 같은 timing requirement를 충족시켜야함
- broadcast time << election timeout << MTBF
	- broadcast time: 서버가 RPC 요청을 병렬적으로 보내고, 이를 받는 평균 시간
	- MTBF (Mean time between failure time):  두 장애가 발생한 평균 시간 간격
- 만약 election timeout이 broadcast time 보다 짧다면 항상 timeout이 발생함
- 만약 election timeout이 MTBF 보다 길다면 시스템은 항상 장애가 발생한 상태에서 복구되지 못함

논문에서는 Election timeout은 150mm ~ 300mm 사이 랜덤한 값으로 설정됨
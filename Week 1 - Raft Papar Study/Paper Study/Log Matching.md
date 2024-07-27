- 만약 두 log의 entry가 같은 index와 term을 갖는다면, 이에 해당하는 명령도 같다
	- Leader는 해당 log index와 term에 최대 하나의 entry만을 생성한다
- 만약 두 log의 entry가 같은 index와 term을 갖는다면, 해당 index와 term 이전의 모든 entry는 동일하다
	- [[AppendEntries]] 과정에서 더 쉽게 두 로그의 일치여부를 판별하기 위해 사용


두 node의 로그 의 일치 여부를 판별하기 위해 사용되는 성질
- Raft 알고리즘의 안전성과, 이해하기 쉬운 로직을 위해 사용됨


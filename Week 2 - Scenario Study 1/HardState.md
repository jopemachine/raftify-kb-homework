--- 
- 서버가 가장 최근에 관측한 term값
- 후보자의 node id
- commit index -> 논문 상에서는 휘발성이지만 코드에서는 hard state
``` Proto
message HardState{
	uint64 term = 1;
	uint64 vote = 2;
	uint64 commit = 3;
}
```
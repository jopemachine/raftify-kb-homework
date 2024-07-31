--- 

```Proto
message Entry{
	EntryType entry_type = 1; // 특수한 엔트리인 것을 알아보기 위해 타입 지정
	uint64 term = 2;
	uint64 index = 3;
	bytes data = 4;
	bytes context = 6;

	bool sync_log=5;
}


message EntryType{
	EntryNormal = 0;
	EntryConfChange = 1;
	EntryConfChangeV2 = 2; // joint Concencus를 지원하면 이용. 두개 이상의 로그엔트리의 상태 변경을 1번에 나타낼 수 .
}
```
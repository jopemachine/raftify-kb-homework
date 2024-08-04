
```
enum EntryType{
    EntryNormal = 0;
    EntryConfChange = 1;
    EntryConfChangeV2 = 2;    // ConfChange which requires joint consesnsus. More than 2 nodes can be changed
}

message Entry{
    EntryType entry_type = 1;
    uint64 term = 2;
    uint64 index = 3;
    bytes data = 4;
    bytes context = 6;
}
```
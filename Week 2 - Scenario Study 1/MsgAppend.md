--- 

노드들끼리 통신할 때 주고 받는 메세지

``` Proto
message Message {
    MessageType msg_type = 1;
    uint64 to = 2;
    uint64 from = 3;
    uint64 term = 4;
    // logTerm is generally used for appending Raft logs to followers.For example,
    // (type=MsgAppend,index=100,log_term=5) means leader appends entries starting at
    // index=101, and the term of entry at index 100 is 5.
    // (type=MsgAppendResponse,reject=true,index=100,log_term=5) means follower rejects some
    // entries from its leader as it already has an entry with term 5 at index 100.
    uint64 log_term = 5;
    uint64 index = 6;
    repeated Entry entries = 7;
    uint64 commit = 8;
    uint64 commit_term = 15;
    Snapshot snapshot = 9;
    uint64 request_snapshot = 13;
    bool reject = 10;
    uint64 reject_hint = 11;
    bytes context = 12;
    uint64 deprecated_priority = 14;
    // If this new field is not set, then use the above old field; otherwise
    // use the new field. When broadcasting request vote, both fields are
    // set if the priority is larger than 0. This change is not a fully
    // compatible change, but it makes minimal impact that only new priority
    // is not recognized by the old nodes during rolling update.
    int64 priority = 16;
}

enum MessageType {
    MsgHup = 0;
    MsgBeat = 1;
    // append entries
    MsgPropose = 2;
    MsgAppend = 3;
    MsgAppendResponse = 4;
    //msg request
    MsgRequestVote = 5;
    MsgRequestVoteResponse = 6;
    
    MsgSnapshot = 7;
    //heartbeat
    MsgHeartbeat = 8;
    MsgHeartbeatResponse = 9;
    MsgUnreachable = 10;
    MsgSnapStatus = 11;
    MsgCheckQuorum = 12;
    MsgTransferLeader = 13;
    MsgTimeoutNow = 14;
    MsgReadIndex = 15;
    MsgReadIndexResp = 16;
    MsgRequestPreVote = 17;
    MsgRequestPreVoteResponse = 18;
}

```
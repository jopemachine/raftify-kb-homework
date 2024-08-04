## How raft gurantee consistency when committing entry

![](https://velog.velcdn.com/images/jollidah/post/e019c88f-210f-4fb1-93ff-59214157120d/image.png)

In Raft, the leader handles inconsistencies by forcing the followers’ logs to duplicate its own.
To bring a follower’s log into consistency with its own, the leader must find the latest log entry where the two logs agree, delete any entries in the follower’s log after that point, and send the follower all of the leader’s entries after that point. The leader maintains a nextIndex for each follower, which is the index of the next log entry the leader will send to that follower. The leader maintains a nextIndex for each follower, which is the index of the next log entry the leader will send to that follower. After a rejection of AppendEntries RPC, the leader decrements nextIndex and retries the AppendEntries RPC. Eventually nextIndex will reach a point where the leader and follower logs match.


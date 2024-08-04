## Election Timeout
A **randomly chosen period** after which a follower assumes the leader has failed and starts a new election. Leader election and ensuring the system remains operational. Raft uses randomized election timeouts to **ensure that split votes are rare** and that they are resolved quickly. To prevent split votes in the first place, election timeouts are chosen randomly from a fixed interval (e.g., 150–300ms).

## Heartbeat Timeout
A shorter, fixed period within which the leader must send a heartbeat to all followers to assert its authority and maintain its leadership. Some time leader send heartbeat intentionally before executing read-only command to check leader itself is valid.

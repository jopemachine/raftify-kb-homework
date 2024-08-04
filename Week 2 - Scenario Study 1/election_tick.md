# election_tick
`election_tick` determines when a follower should start a new leader election if it hasn’t received a heartbeat from the leader.

- **Timeout**: If no heartbeat is received within `election_tick`, the follower initiates an election.
- **Randomization**: `election_tick` is randomly set between `min_election_tick` and `max_election_tick` to prevent simultaneous elections and vote splits.

## Process
1. **Heartbeat Missed**: Follower detects a timeout and starts an election.
2. **Become Candidate**: The follower sends `MsgRequestVote` messages to other nodes.
3. **Election Outcome**: The candidate with the smallest `election_tick` value is more likely to be elected if it gets a majority of votes.

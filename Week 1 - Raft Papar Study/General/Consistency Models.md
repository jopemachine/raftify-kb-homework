# Consistency Models in Raft
Raft emphasizes strong consistency through a structured approach to `leader election`, `log replication`.

# Strong Consistency
Ensures that all nodes in the system reflect the same state at any given time after a consensus decision.

- **Leader Election**: A **single leader** is elected to coordinate updates. Only the leader can propose log entries.
- **Log Replication**: The leader **replicates log entries** to the followers. An entry is considered **committed when a majority of the nodes have written** it to their logs.
- **Commitment**: Once an entry is committed, it is guaranteed to be durable and will be applied to the state machine.
- **Linearizability**: All of commands flow from leader to follower. This flow prevent confusion.

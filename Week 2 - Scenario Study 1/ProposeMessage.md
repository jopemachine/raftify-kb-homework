## Propose Message

1. **Request Generation**: A client initiates a request by calling methods like `RaftServiceClient.propose()` or `RaftNode.propose()`.
2. **gRPC Communication**: This request is relayed to the remote Raft node's `RaftServiceClient.propose()` via gRPC.
3. **Channel Transmission**: The `RaftServiceClient.propose()` method sends a `Propose` message to the `RaftNode.run()` asynchronous task through a channel.
4. **Message Queue Polling**: The `RaftNode.run()` task, which polls the message queue, receives the `Propose` message and subsequently calls `RawNode.propose()`.
5. **State Machine Update**: If there are changes that need to be applied to the state machine, a `Ready` instance is created and passed to the `on_ready()` handler.
6. **Handling Commit**: The `on_ready()` handler processes the committed entries and then responds to the client.

- **Propose Message**: This term refers to a message type defined to propose state changes to the cluster. It initiates the process of replicating and committing new log entries across the Raft nodes.

By following this flow, Raft ensures that client requests are properly handled, state changes are proposed, and eventually committed to maintain consistency across the distributed system.

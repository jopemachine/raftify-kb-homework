#### Sync
- The caller waits for the callee to finish its execution.

#### Async
- Caller **doesn't wait** for the callee's reponse and proceeds with the next task.
- The caller is **indifferent** to the callee, with no defined response time.
- **Two Generals' Problem**: In extreme cases, two generals can never reach consensus if all requests and responses are lost.
- **FLP Impossibility**: A perfectly asynchronous distributed system that makes no timing assumptions cannot exist.
- At some point, the system must **wait for a response synchronously** due to timing assumptions.

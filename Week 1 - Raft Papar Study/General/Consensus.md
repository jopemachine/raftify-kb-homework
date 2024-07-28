# Consensus in Distributed Systems
Consensus in distributed systems refers to the process by which multiple nodes agree on a single data value or course of action, ensuring system reliability and consistency even in the presence of faults.

Factors that make consensus more robust
- **Partition Tolerance**: The system's ability to restore functionality after a connection loss.
- **Availability**: The percentage of time the system can respond (e.g., 5m 15s downtime per year -> 99.999% availability).
- **Consistency**: How consistently the system responds "Normally".
	- Consistency Models: How can we define "Normally".
	- Sequential Consistency Model, Causal Consistency Model, **Eventual Consistency Model**, **Strong Consistency Model**...

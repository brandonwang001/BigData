*Raft is a consensus algorithm for managing a replicated log. It produces a result equivalent to **(multi-)Paxos**, and it is as efficient as Paxos, but its structure is different from Paxos; this makes Raft more understandable than Paxos and also provides a better foundation for build- ing practical systems. In order to enhance understandabil- ity, Raft separates the key elements of consensus, such as **leader election, log replication, and safety,** and it enforces a stronger degree of coherency to reduce the number of states that must be considered. Results from a user study demonstrate that Raft is easier for students to learn than Paxos. Raft also includes a new mechanism for changing the cluster membership, which uses overlapping majori- ties to guarantee safety.*

#### NOTES:
1. raft等效于multi-paxos。
2. raft一直协议由多个独立的部分组成：**选主**、**日志复制**和**安全**。

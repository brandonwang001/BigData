> Raft is a consensus algorithm for managing a replicated log. It produces a result equivalent to **(multi-)Paxos**, and it is as efficient as Paxos, but its structure is different from Paxos; this makes Raft more understandable than Paxos and also provides a better foundation for build- ing practical systems. In order to enhance understandabil- ity, Raft separates the key elements of consensus, such as **leader election, log replication, and safety,** and it enforces a stronger degree of coherency to reduce the number of states that must be considered. Results from a user study demonstrate that Raft is easier for students to learn than Paxos. Raft also includes a new mechanism for changing the cluster membership, which uses overlapping majori- ties to guarantee safety.
> #### NOTES:
> 1. raft等效于multi-paxos。
> 2. raft一直协议由多个独立的部分组成：**选主**、**日志复制**和**安全**。

> Consensus algorithms allow a collection of machines to work as a coherent group that can survive the fail- ures of some of its members. Because of this, they play a key role in building reliable large-scale software systems. Paxos [15, 16] has dominated the discussion of consen- sus algorithms over the last decade: most implementations of consensus are based on Paxos or influenced by it, and Paxos has become the primary vehicle used to teach stu- dents about consensus.
> #### NOTES:
> 1. 一致性协议是分布式的基石。
> 2. Paxos大有一统天下之势。

> *Unfortunately, Paxos is quite difficult to understand, in spite of numerous attempts to make it more approachable. Furthermore, its architecture requires complex changes to support practical systems. As a result, both system builders and students struggle with Paxos.*
> #### NOTES:
> 1. Paxos很难懂。（两片论文看了多遍，还是了解流程。）

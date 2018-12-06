> *Raft is a consensus algorithm for managing a replicated log. It produces a result equivalent to **(multi-)Paxos**, and it is as efficient as Paxos, but its structure is different from Paxos; this makes Raft more understandable than Paxos and also provides a better foundation for build- ing practical systems. In order to enhance understandabil- ity, Raft separates the key elements of consensus, such as **leader election, log replication, and safety,** and it enforces a stronger degree of coherency to reduce the number of states that must be considered. Results from a user study demonstrate that Raft is easier for students to learn than Paxos. Raft also includes a new mechanism for changing the cluster membership, which uses overlapping majori- ties to guarantee safety.*
> > #### NOTES:
> > 1. raft等效于multi-paxos。
> > 2. raft一直协议由多个独立的部分组成：**选主**、**日志复制**和**安全**。

> **Strong leader:** Raft uses a stronger form of leader- ship than other consensus algorithms. For example, log entries only flow from the leader to other servers. This simplifies the management of the replicated log and makes Raft easier to understand.
> > #### NOTE:
> > 1.日志只能从主机拷贝到备机。（和Paxos有什么区别？）

> **Leader election:** Raft uses randomized timers to elect leaders. This adds only a small amount of mechanism to the heartbeats already required for any consensus algorithm, while resolving conflicts sim- ply and rapidly.
> > #### NOTES:
> > 1.通过简单改造心跳协议，raft采用了**随机计时器**来选主，从而可以快速简单的坚决选注冲突的问题。

> **Membership changes:** Raft’s mechanism for changing the set of servers in the cluster uses a new joint consensus approach where the majorities of two different configurations overlap during transi- tions. This allows the cluster to continue operating normally during configuration changes.
> > #### NOTES:
> > 1. 为了解决成员变更的问题，raft使用了一种**联合一致方法**（使用两个配置重叠的大多数机器）。这种方法可以保证在集群变更的时候也可以正常操作。

> **Replicated state machines**
> Consensus algorithms typically arise in the context of replicated state machines [37]. In this approach, state ma- chines on a collection of servers compute identical copies of the same state and can continue operating even if some of the servers are down. Replicated state machines are used to solve a variety of fault tolerance problems in dis- tributed systems. For example, large-scale systems that have a single cluster leader, such as GFS [8], HDFS [38], and RAMCloud [33], typically use a separate replicated state machine to manage leader election and store config- uration information that must survive leader crashes. Ex- amples of replicated state machines include Chubby [2] and ZooKeeper [11].
> > #### NOTES:
> > 1. 一般提到一致性协议便会联系到**复制状态机**。
> > 2. 复制状态机通常被用来解决分布式系统中的容错问题。
> > 3. 在分布式大规模系统中存在master节点的，需要使用**复制状态机来管理选主**和存储的相关配置信息。
> > 4. 使用复制状态机的典型系统如：chubby和zookeeper。

> Replicated state machines are typically implemented using a replicated log, as shown in Figure 1. Each server stores a log containing a series of commands, which its state machine executes in order. Each log contains the same commands in the same order, so each state ma- chine processes the same sequence of commands. Since the state machines are deterministic, each computes the same state and the same sequence of outputs.
> ![Replicated state machines](./images/replicated_state_machines.png)
> > #### NOTES:
> > 1. 复制状态机的实现通常是通过使用复制日志来实现的。
> > 2. 每台机器的状态机具有相同的起始状态，另外每台机器都有一个存储操作的日志文件，操作序列在每台机器上都有相同的顺序。所有的操作按顺序在扭转状态机，则可以保证每台机器都有相同的输出序列。

> They ensure safety (never returning an incorrect re- sult) under all non-Byzantine conditions, including network delays, partitions, and packet loss, duplica- tion, and reordering.

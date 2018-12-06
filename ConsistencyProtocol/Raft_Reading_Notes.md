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
> > #### NOTES:
> > 1. 在非拜占庭条件下，一致性协议应该保证在网络延迟、分区、丢包、重复和乱序情况下返回正确结果。
> > 2. 拜占庭将军问题（Byzantine failures），是由莱斯利·兰伯特提出的点对点通信中的基本问题。含义是在存在消息丢失的不可靠信道上试图通过消息传递的方式达到一致性是不可能的。因此对一致性的研究一般假设信道是可靠的，或不存在本问题。

> They are fully functional (available) as long as any majority of the servers are operational and can com- municate with each other and with clients. Thus, a typical cluster of five servers can tolerate the failure of any two servers. Servers are assumed to fail by stopping; they may later recover from state on stable storage and rejoin the cluster.
> > #### NOTES:
> > 1. 一致性协议需要保证在大多数服务器存活的情况下仍可以正常工作，不可服务的机器恢复并加入到集群中。

> They do not depend on timing to ensure the consis-tency of the logs: faulty clocks and extreme message delays can, at worst, cause availability problems.
> > #### NOTES:
> > 1. 一致性协议并不通过时间来保证一致。
> > 2. 如果通过时间来保证日志一致，那么错误的时钟、严重的网络延迟都会导致可用性问题。

> In the common case, a command can complete as soon as a majority of the cluster has responded to a single round of remote procedure calls; a minority of slow servers need not impact overall system perfor- mance.
> > #### NOTES:
> > 1. 尽量在一轮RPC后保证集群的大多数快速相应。
> > 2. 不能因为少数慢速服务降低整个系统的性能。

> Over the last ten years, Leslie Lamport’s Paxos proto- col [15] has become almost synonymous with consensus: it is the protocol most commonly taught in courses, and most implementations of consensus use it as a starting point. Paxos first defines a protocol capable of reaching agreement on a single decision, such as a single replicated log entry. We refer to this subset as single-decree Paxos. Paxos then combines multiple instances of this protocol to facilitate a series of decisions such as a log (multi-Paxos). Paxos ensures both safety and liveness, and it supports changes in cluster membership. Its correctness has been proven, and it is efficient in the normal case.
> > #### NOTES:
> > 1. Paxos最初提出了对于单次提议达成一致的协议。 *single-decree Paxos* 
> > 2. 基于单次的一致性协议，通过多个实例来达成一系列提议。*multi-Paxos*
> > 3. Paxos保证安全、生存，并且支持集群成员变更。并且正确性是经过证明的，在正常情况下都是有效的。

> Unfortunately, Paxos has two significant drawbacks. The first drawback is that Paxos is exceptionally diffi- cult to understand. The full explanation [15] is notori- ously opaque; few people succeed in understanding it, and only with great effort. As a result, there have been several attempts to explain Paxos in simpler terms [16, 20, 21]. These explanations focus on the single-decree subset, yet they are still challenging. In an informal survey of atten- dees at NSDI 2012, we found few people who were com- fortable with Paxos, even among seasoned researchers. We struggled with Paxos ourselves; we were not able to understand the complete protocol until after reading sev- eral simplified explanations and designing our own alter- native protocol, a process that took almost a year.
> > #### NOTES:
> > 1. Paxos第一个缺点就是艰涩难懂，即便是single-decree Paxos。

> We hypothesize that Paxos’ opaqueness derives from its choice of the single-decree subset as its foundation. Single-decree Paxos is dense and subtle: it is divided into two stages that do not have simple intuitive explanations and cannot be understood independently. Because of this, it is difficult to develop intuitions about why the single- decree protocol works. The composition rules for multi- Paxos add significant additional complexity and subtlety. We believe that the overall problem of reaching consensus on multiple decisions (i.e., a log instead of a single entry) can be decomposed in other ways that are more direct and obvious.
> > #### NOTES:
> > 1. single-decree Paxos之所以晦涩难懂，因为协议本身设计的紧凑精巧导致不能进行拆分理解。

> The second problem with Paxos is that it does not pro- vide a good foundation for building practical implemen- tations. One reason is that there is no widely agreed- upon algorithm for multi-Paxos. Lamport’s descriptions are mostly about single-decree Paxos; he sketched possi- ble approaches to multi-Paxos, but many details are miss- ing. There have been several attempts to flesh out and op- timize Paxos, such as [26], [39], and [13], but these differ from each other and from Lamport’s sketches. Systems such as Chubby [4] have implemented Paxos-like algo- rithms, but in most cases their details have not been pub- lished.
> > #### NOTES:
> > 1. Paxos的第二确定缺点是工程实现中没有大家都赞同的multi-Paxos的算法。存在很多multi-Paxos的不同实现，且大多细节是不公开的。

> Furthermore, the Paxos architecture is a poor one for building practical systems; this is another consequence of the single-decree decomposition. For example, there is lit- tle benefit to choosing a collection of log entries indepen- dently and then melding them into a sequential log; this just adds complexity. It is simpler and more efficient to design a system around a log, where new entries are ap- pended sequentially in a constrained order. Another prob- lem is that Paxos uses a symmetric peer-to-peer approach at its core (though it eventually suggests a weak form of leadership as a performance optimization). This makes sense in a simplified world where only one decision will be made, but few practical systems use this approach. If a series of decisions must be made, it is simpler and faster to first elect a leader, then have the leader coordinate the decisions.
> As a result, practical systems bear little resemblance to Paxos. Each implementation begins with Paxos, dis- covers the difficulties in implementing it, and then de- velops a significantly different architecture. This is time- consuming and error-prone, and the difficulties of under- standing Paxos exacerbate the problem. Paxos’ formula- tion may be a good one for proving theorems about its cor- rectness, but real implementations are so different from Paxos that the proofs have little value. The following com- ment from the Chubby implementers is typical:
> There are significant gaps between the description of the Paxos algorithm and the needs of a real-world system. . . . the final system will be based on an un- proven protocol [4].
> Because of these problems, we concluded that Paxos does not provide a good foundation either for system building or for education. Given the importance of con- sensus in large-scale software systems, we decided to see if we could design an alternative consensus algorithm with better properties than Paxos. Raft is the result of that experiment.
> > #### NOTES:
> > 1. 这么多内容就是讲Paxos从论文到实际系统还是异常困难的。
> > 2. 据说微信后台在开发Paxos协议时，三个人独立理解论文进行开发，并行测试三个人的实现，确认三个人理解一致后合同开发了最终版本。(^-^)，对于一个业余小白想独立开发一个生产环境的Paxos要上天啊。

> We had several goals in designing Raft: it must provide a complete and practical foundation for system building, so that it significantly reduces the amount of design work required of developers; it must be safe under all conditions and available under typical operating conditions; and it must be efficient for common operations. But our most important goal—and most difficult challenge—was un- derstandability. It must be possible for a large audience to understand the algorithm comfortably. In addition, it must be possible to develop intuitions about the algorithm, so that system builders can make the extensions that are in- evitable in real-world implementations.
> > ####NOTES:
> > 1. Raft的提出就是为了解决Paxos的晦涩难懂和工程实现问题。(Leslie Lamport大神的文章都是高屋建瓴)
> > 2. [Leslie Lamport Home Page](http://www.lamport.org)，看了下大佬的论文，183片文章，是著作等身的最好注解。

> We recognize that there is a high degree of subjectiv- ity in such analysis; nonetheless, we used two techniques that are generally applicable. The first technique is the well-known approach of problem decomposition: wher- ever possible, we divided problems into separate pieces that could be solved, explained, and understood relatively independently. For example, in Raft we separated leader election, log replication, safety, and membership changes.
> Our second approach was to simplify the state space by reducing the number of states to consider, making the system more coherent and eliminating nondeterminism where possible. Specifically, logs are not allowed to have holes, and Raft limits the ways in which logs can become inconsistent with each other. Although in most cases we tried to eliminate nondeterminism, there are some situ- ations where nondeterminism actually improves under- standability. In particular, randomized approaches intro- duce nondeterminism, but they tend to reduce the state space by handling all possible choices in a similar fashion (“choose any; it doesn’t matter”). We used randomization to simplify the Raft leader election algorithm.
> > #### NOTES:
> > 1. 为了设计便于理解的算法，作者运用了分解问题和减少状态机的方法降低理解的复杂度。
> > 2. Raft使用了随机的方法来简化选主。

> Raft is an algorithm for managing a replicated log of the form described in Section 2. Figure 2 summarizes the algorithm in condensed form for reference, and Figure 3 lists key properties of the algorithm; the elements of these figures are discussed piecewise over the rest of this sec- tion.
> ![Raft State](./images/raft_state.png)
> > #### NOTES:
> > - **Persistent state on all servers**
> >     - 1. 持久的状态包含：currentTerm 、 votedFor 和 log[], 上面的状态会在进行rpc之前进行稳定存储介质的持久化。
> >     - 2. currentTerm : 当前服务可见的最新任期，初始为0且单调。
> >     - 3. votedFor : 当前任期已接受的候选ID。
> >     - 4. log[] : 日志记录，每个记录包含状态机的操作命令和被Leader接受的任期(索引初始化为1)。
> > - **Persistent state on all servers**
> >     - 1. commitIndex : 已知最高的已被提交的日志记录的索引（初始化为0，单调递增）。
> >     - 2. lastApplied : 已知最高的已被应用到状态记得日志记录的索引。
> > - **Volatile state on leaders**
> >     - 1. 选举后所有状态重新初始化。
> >     - 2. nextIndex[] : 存储发往每个server的下条日志的索引。(初始化为leader最后一条日志的索引+1)
> >     - 3. matchIndex[] : 存储将进行复制的日志记录的索引。(初始化为0，单调递增)

> ![AppendEntries RPC](./images/append_entries_rpc.png) 
> > #### NOTES:
> > - Leader进行复制日志和心跳。
> > - **Arguments**
> >     - **term** : leader的任期
> >     - **leaderId** : follower重定向请求到leader。
> >     - **prevLogIndex** : 最新一条日志的前继日志的索引。
> >     - **prevLogTerm** : 前继日志的任期。
> >     - **entries[]** : 日志存储。（为空时进行心跳探测）
> >     - **leaderCommit** : leader已提交的最大日志索引。
> > - **Results:**
> >     - **term** : 当前任期，Leader更新自己。
> >     - **success** : 如果follower包含和prevLogIndex、prevLogTerm匹配的日志
> > - **Receiver implementation**
> >     1. term < currentTerm ? false : true;
> >     2. 如果日志不包含和prevLogIndex、prevLogTerm匹配的日志返回false
> >     3. 如果存在和最新任期不同的日志，删除存在的日志，其他follower追赶它。
> >     4. 如果日志之前不存在，则直接存储
> >     5. if leaderCommit > commitIndex, set commitIndex = min(leaderCommit, index of last new entry)。

> ![RequestVote RPC](./images/request_vote_rpc.png)
> > - 候选者发起获取选票。
> > - **Arguments**
> >     - **term** : 候选者任期
> >     - **candidateId** : 请求选票的候选者id。
> >     - **lastLogIndex** : 候选者最新一条日志的索引。
> >     - **lastLogTerm** : 候选者最新一条日志的任期。
> > - **Results**
> >     - **term** : 当前任期，用于候选者更新自己
> >     - **voteGranted** : 如果候选者获得选票，其值为真。
> > - **Receiver implementation**
> >     1. Reply false if term < currentTerm
> >     2. 如果当前任期已接受的候选为NULL或者等于请求选票的候选者id，并且候选的日志和接受者的日志保持一致，则获得当前选票。

> ![All Servers](./images/all_servers.png)
> > - **All Sercers** 
> >     1. 已提交的日志索引大于已应用的日志索引，增加已应用的日志索引，将log[lastApplied]应用到状态级。
> >     2. 如果请求rpc或者相应rpc包含任期大于currentTerm的任期T，设置currentTerm = T，切换到follower。

> ![Followers](./images/followers.png)
> > - **Followers**
> >     1. 响应候选者或者leader的rpc。
> >     2. 如果选举超时内，没有接受到leader的AppendEntries, 同时也没有接受到候选者的投票请求，自己则转化为候选者。

> ![Candidates](./images/candidates.png)
> > - **Candidates**
> >     1. 转换为候选者，开始进行选举。
> >         - 增加当前任期
> >         - 选举自己
> >         - 重置选举计时器
> >         - 发送选举rpc请求到其他的服务器
> >     2. 如果得到大多数机器的投票，则转变化leader。
> >     3. 如果接受搭配追加日志的rpc请求，则转变为follower
> >     4. 如果选举计时器超时，则重新进行选举。

> ![Leaders](./images/leaders.png)
> > - **Leaders**
> >     1. 一旦当选为leader，发送空的追加日志rpc（心跳）请求到每个服务器，在空闲的事件发送心跳到其他机器，延长自己的任期。
> >     2. 如果接受到client的请求，追加日志到本地日志，当日志被执行到转台机后回应客户端。
> >     3. 如果一个追随者的日志索引大于下一条日志索引，发送追加日志的rpc请求，日志的索引是从下一跳日志索引开始。
> >         - 如果成功，更新follower的下一跳日志索引和匹配日志索引。
> >         - 如果因为不一致失败，减小下一跳日志索引并重试。
> >     4. 如果存在N,并且N大于已提交日志索引，同时匹配索引的大多数大于N, 并且这条日志的任期等于当前任期，设置已提交的日志索引为N。



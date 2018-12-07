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

> ![Raft guarantees Property](./images/property.png)
> > - **Properties**
> >     - Election Safety : 在一个选举周内最多被选举出一个Leader。
> >     - Leader Append-Only : 对于一个Leader永远不覆盖写和删除日志，只追加日志。
> >     - Log Matching : 如果日志包含两条entry有相同的索引和任期，那么日志是唯一在所有的跟定索引。
> >     - Leader Completeness : 如果一条日志记录在一个跟定的任期内已经提交，对于所有大于当前任期的任期，这条日志都是存在的。
> >     - State Machine Safety : 如果一个服务器应用一条跟定索引的日志到到状态机，那么其他所有的机器不允许应用一个有相同索引但内容不同的日志。

> Raft is an algorithm for managing a replicated log of the form described in Section 2. Figure 2 summarizes the algorithm in condensed form for reference, and Figure 3 lists key properties of the algorithm; the elements of these figures are discussed piecewise over the rest of this sec- tion.
> Raft implements consensus by first electing a distin- guished leader, then giving the leader complete responsi- bility for managing the replicated log. The leader accepts log entries from clients, replicates them on other servers, and tells servers when it is safe to apply log entries to their state machines. Having a leader simplifies the man- agement of the replicated log. For example, the leader can decide where to place new entries in the log without con- sulting other servers, and data flows in a simple fashion from the leader to other servers. A leader can fail or be- come disconnected from the other servers, in which case a new leader is elected.
> > #### NOTES:
> > 1. 首先选出一个Leader。
> > 2. Leader具有控制状态机的责任。
> > 3. Leader从客户端接受日志记录，并发日志记录复制到其他server。
> > 4. 通知其他servers，当日志记录被安全的应用到状态机。
> > 5. 如果有一个Leader可以简化复制日志的管理。

> Given the leader approach, Raft decomposes the con- sensus problem into three relatively independent subprob- lems, which are discussed in the subsections that follow:
> > #### NOTES:
> > 1. Raft的一致性保证可以分解为3个子问题：选主、日志复制和安全。

> Leader election: a new leader must be chosen when an existing leader fails (Section 5.2).
> > #### NOTES:
> > 1. 当存在Leader失败的时候，必须选举一个新的Leader。

> Log replication: the leader must accept log entries from clients and replicate them across the cluster, forcing the other logs to agree with its own (Sec- tion 5.3).
> > #### NOTES:
> > 1. Leader必须接受client的日志请求，并且保证将日志复制到集群的其他机器。

> Safety: the key safety property for Raft is the State Machine Safety Property in Figure 3: if any server has applied a particular log entry to its state machine, then no other server may apply a different command for the same log index. Section 5.4 describes how Raft ensures this property; the solution involves an additional restriction on the election mechanism de- scribed in Section 5.2.
> > #### NOTES:
> > 1. 安全性的重点是状态机的安全，如果某台机器应用一条日志到状态机，那么其他机器也必须是相同的日志，Raft通过附加的约束来保证。

> A Raft cluster contains several servers; five is a typical number, which allows the system to tolerate two failures. At any given time each server is in one of three states: leader, follower, or candidate. In normal operation there is exactly one leader and all of the other servers are fol- lowers. Followers are passive: they issue no requests on their own but simply respond to requests from leaders and candidates. The leader handles all client requests (if a client contacts a follower, the follower redirects it to the leader). The third state, candidate, is used to elect a new leader as described in Section 5.2. Figure 4 shows the states and their transitions; the transitions are discussed below.
> > #### NOTES:
> > 1. N = 2F + 1,应用Raft协议可以保证在F机器不能工作情况下，仍然能保证一致正常工作。
> > 2. 一台机器可能处于三种状态中的一种：Leader、Follower和Candidate。
> > 3. 正常状态下一台为Leader，其它为Followder。
> > 4. Follower是被动的，只是回应Leader和Candidate的请求。
> > 5. 所有客户端的请求都有Leader来负责处理，如果请求到达Follower，则会被重定向到Leader。

> ![Server State](./images/server_state.png)
> > #### NOTES:
> > 1. 起始所有的机器均为Follower。
> > 2. 如果Follower的计时器超时，则Follwer转化自己为Candidate。
> > 3. 如果Candidate计时器超时，此时还没够Leader则继续作为Candidate。
> > 4. 如果Candidate发现选举出Leader，或者一个新的任期开始。则转化为Follower。
> > 5. 如果Candidate接受到大多Follower的选票，那么转化为Leader。
> > 6. 如果发现有更高的任期存在，则转化为Follower。

> Raft divides time into terms of arbitrary length, as shown in Figure 5. Terms are numbered with consecutive integers. Each term begins with an election, in which one or more candidates attempt to become leader as described in Section 5.2. If a candidate wins the election, then it serves as leader for the rest of the term. In some situations an election will result in a split vote. In this case the term will end with no leader; a new term (with a new election)will begin shortly. Raft ensures that there is at most one leader in a given term.
> > #### NOTES:
> > 1. Raft把事件切片为不同长度的任期。
> > 2. 每个任期有一个编号，编号为连续的整数。
> > 3. 每个任期的开始都是从选举开始的，此时有一个或者多个server尝试成为Leader。
> > 4. 一旦一个候选者在选举中获胜，这个任期内剩下的事件，这个server就作为Leader。
> > 5. 肯能存在某个任期没有成功选出Leader，那么下个任期将会开始。
> > 6. Raft保证每个任期内被选举出的Leader不多余一个。
> > ![Term](./images/term.png)

> Different servers may observe the transitions between terms at different times, and in some situations a server may not observe an election or even entire terms. Terms act as a logical clock [14] in Raft, and they allow servers to detect obsolete information such as stale leaders. Each server stores a current term number, which increases monotonically over time. Current terms are exchanged whenever servers communicate; if one server’s current term is smaller than the other’s, then it updates its current term to the larger value. If a candidate or leader discovers that its term is out of date, it immediately reverts to fol- lower state. If a server receives a request with a stale term number, it rejects the request.
> > #### NOTES:
> > 1. 任期在Raft中扮演这逻辑锁的概念, 他们允许每个server获取leeder的信息。
> > 2. 每个机器存在一个单调递增的current term number。
> > 3. 当机器交互时，会进行当前任期的交换。
> > 4. 如果一个机器的任期小于其它机器的任期，它会将自己任期更新为最大的任期。
> > 5. 如果一个候选者或者leader发现自己的任期过期，则会扭转到follower状态。
> > 6. 如果一台机器接受到一个老任期的请求，则直接拒绝。

> Raft servers communicate using remote procedure calls (RPCs), and the basic consensus algorithm requires only two types of RPCs. RequestVote RPCs are initiated by candidates during elections (Section 5.2), and Append- Entries RPCs are initiated by leaders to replicate log en- tries and to provide a form of heartbeat (Section 5.3). Sec- tion 7 adds a third RPC for transferring snapshots between servers. Servers retry RPCs if they do not receive a re- sponse in a timely manner, and they issue RPCs in parallel for best performance.
> > #### NOTES:
> > 1. Raft的机器之间是通过rpc来进行通信，rpc只需RequestVote RPCs 和Append- Entries RPCs。
> > 2. RequestVote RPCs 是candidate在候选期间使用的。
> > 3. Append- Entries RPCs 是复制日志和保持心跳。
> > 4. 另外一种rpc用来交换机器的快照。
> > 5. 对于重试请求可以通过并发可以获得更好的性能。

> Raft uses a heartbeat mechanism to trigger leader elec- tion. When servers start up, they begin as followers. A server remains in follower state as long as it receives valid RPCs from a leader or candidate. Leaders send periodic heartbeats (AppendEntries RPCs that carry no log entries) to all followers in order to maintain their authority. If a follower receives no communication over a period of time called the election timeout, then it assumes there is no vi- able leader and begins an election to choose a new leader.
> > #### NOTES:
> > 1. Raft使用心跳机制来触发选主。
> > 2. 机器启动是都是Follower，如果一台机器接受到来自Leader或者candidate的合法请求，它就保持在Follower状态。
> > 3. Leader通过发送周期性心跳来保证自己的权限。
> > 4. 如果选举定时器超时，则发起选举进行新一个任期的选举。 

> To begin an election, a follower increments its current term and transitions to candidate state. It then votes for itself and issues RequestVote RPCs in parallel to each of the other servers in the cluster. A candidate continues in this state until one of three things happens: (a) it wins the election, (b) another server establishes itself as leader, or (c) a period of time goes by with no winner. These out- comes are discussed separately in the paragraphs below.
> > #### NOTES:
> > 1. 选举开始，追随者增加自己的当前任期，并且转变为候选者状态。 
> > 2. 然后候选者发起请求，推荐选举自己，通过并发RequestVote RPCs到集群中。
> > 3. 出现三种情况，将导致候选者的状态发生迁移:
> >     - 赢得选举
> >     - 有server证实自己获得选举
> >     - 或者一段事件后，集群中没有人获得选举

> A candidate wins an election if it receives votes from a majority of the servers in the full cluster for the same term. Each server will vote for at most one candidate in a given term, on a first-come-first-served basis (note: Sec- tion 5.4 adds an additional restriction on votes). The ma- jority rule ensures that at most one candidate can win the election for a particular term (the Election Safety Prop- erty in Figure 3). Once a candidate wins an election, it becomes leader. It then sends heartbeat messages to all of the other servers to establish its authority and prevent new elections.
> > #### NOTES:
> > 1. 当一个候选者在一个任期内，获得集群中大多数的机器的选票，则候选者在选举中胜出成为leader。
> > 2. 在一个任期内，一个server最多选举一个候选者，并且遵守先到先服务的原则。
> > 3. 众多的规则保证在一个特定的周期内，最多只有一个候选者可以选举胜出。
> > 4. 一旦Candidate获得选举胜出成为Leader，它将发送心跳到集群的其他机器确认自己的授权，并组织发起新的选举。

> While waiting for votes, a candidate may receive an AppendEntries RPC from another server claiming to be leader. If the leader’s term (included in its RPC) is at least as large as the candidate’s current term, then the candidate recognizes the leader as legitimate and returns to follower state. If the term in the RPC is smaller than the candidate’s current term, then the candidate rejects the RPC and con- tinues in candidate state.
> > #### NOTES:
> > 1. 当候选者处在等待选票的状态时，候选者如果接受到来自其他机器的请求，并声称自己是leader，只有当leader的任期大于等于自己本地任期，才认为当前leader是合法的并迁移到Follower的状态。否则候选者可以拒绝假Leader的请求，并保持在候选者状态。

> The third possible outcome is that a candidate neither wins nor loses the election: if many followers become candidates at the same time, votes could be split so that no candidate obtains a majority. When this happens, each candidate will time out and start a new election by incre- menting its term and initiating another round of Request- Vote RPCs. However, without extra measures split votes could repeat indefinitely.
> > #### NOTES:
> > 1. 第三种的可能是一个候选者既没有获胜也没有失败。（选举流产了）
> > 2. 选举流产的发生因为当所有Follower同时迁移到候选者状态，并同时向集群的其他机器发去选举的rpc请求。导致没有任何一个机器可以获得集群的多数派的选举。
> > 3. 当上面的这种情况发生，每个候选者将会超时，然后会发起新一轮的选举。
> > 4. 为了避免上述情况的发生，需要添加额外的约束，来保证不会无限期的发生。

> Raft uses randomized election timeouts to ensure that split votes are rare and that they are resolved quickly. To prevent split votes in the first place, election timeouts are chosen randomly from a fixed interval (e.g., 150–300ms). This spreads out the servers so that in most cases only a single server will time out; it wins the election and sends heartbeats before any other servers time out. The same mechanism is used to handle split votes. Each candidate restarts its randomized election timeout at the start of an election, and it waits for that timeout to elapse before starting the next election; this reduces the likelihood of another split vote in the new election. Section 9.3 shows that this approach elects a leader rapidly.
> > #### NOTES:
> > 1. Raft使用随机选举超时来解决选举分裂的情况并快速解决问题。
> > 2. 为了避免选举分裂的发生，选举发生时每个server随机选择一个选举超时时间[150,300]ms。
> > 3. 上面的随机选举超时可以保证在大多数情况下，只有一台Server发生超时。
> > 4. 选举超时的server可以在其他server还没有超时的情况下，选举胜出并发送心跳。
> > 5. 相同的机制来被用来防止选举分裂情况的发生。
> > 6. 每个候选者在启动选举的时候随机选择一个选举超时时间，这样可以降低下一个选举周期发生选举分裂的概率。
> > 7. Raft表明这样的选举是非常有效的。

> Elections are an example of how understandability guided our choice between design alternatives. Initially we planned to use a ranking system: each candidate was assigned a unique rank, which was used to select between competing candidates. If a candidate discovered another candidate with higher rank, it would return to follower state so that the higher ranking candidate could more eas- ily win the next election. We found that this approach created subtle issues around availability (a lower-ranked server might need to time out and become a candidate again if a higher-ranked server fails, but if it does so too soon, it can reset progress towards electing a leader). We made adjustments to the algorithm several times, but after each adjustment new corner cases appeared. Eventually we concluded that the randomized retry approach is more obvious and understandable.
> > #### NOTES:
> > 1. 作者提出的随机选主法是经过多次迭代和探索得出一个直观且易于理解的方案。 
 

---
title: raft实现
date: '2024-07-24 20:56:36'
updated: '2024-08-18 19:36:06'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1721826081013-792a578c-813a-43cc-b927-1ed23faaa927.png'
description: raft论文介绍主要流程图图 1 ：复制状态机的结构。一致性算法管理着来自客户端指令的复制日志。状态机从日志中处理相同顺序的相同指令，所以产生的结果也是相同的。通过一致性传送log，发送指令，最终达到状态机一致关键词followercandidateleader只有candidate才可以变成...
---




# raft论文介绍
主要流程图

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ad699177fffe87666bf7d433c6577974.png?token=AGECE2IEEWHNOEQKMS7ESSLG72WXM)



> <font style="color:rgb(99, 108, 118);">图 1 ：复制状态机的结构。一致性算法管理着来自客户端指令的复制日志。状态机从日志中处理相同顺序的相同指令，所以产生的结果也是相同的。</font>
>

:::info
通过一致性传送log，发送指令，最终达到状态机一致

:::

关键词

1. follower
2. candidate
3. leader

只有candidate才可以变成leader，follower发送心跳，找不到leader，自己尝试开始变成candidate，然后发起新的一轮选举，选举得票数超过一半，才可以变成leader。（注意判断如何得到票数，是根据term任期的大小来决定的，如果对方的term大于当前自我的term，我就有candidate变成follower

:::info
这就有三个函数，becomeleader，becomecandidate，becomefollo

:::



---

## raft解决思路
1. 领导选举
2. 日志复制
3. 一致性安全

**<font style="color:rgb(31, 35, 40);"></font>**

**<font style="color:rgb(31, 35, 40);">状态</font>**<font style="color:rgb(31, 35, 40);">：所有服务器上的持久性状态 (在响应 RPC 请求之前，已经更新到了稳定的存储设备)</font>

| <font style="color:rgb(31, 35, 40);">参数</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">currentTerm</font> | <font style="color:rgb(31, 35, 40);">服务器已知最新的任期（在服务器首次启动时初始化为0，单调递增）</font> |
| <font style="color:rgb(31, 35, 40);">votedFor</font> | <font style="color:rgb(31, 35, 40);">当前任期内收到选票的 candidateId，如果没有投给任何候选人 则为空</font> |
| <font style="color:rgb(31, 35, 40);">log[]</font> | <font style="color:rgb(31, 35, 40);">日志条目；每个条目包含了用于状态机的命令，以及领导人接收到该条目时的任期（初始索引为1）</font> |


<font style="color:rgb(31, 35, 40);">所有服务器上的易失性状态</font>

| <font style="color:rgb(31, 35, 40);">参数</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">commitIndex</font> | <font style="color:rgb(31, 35, 40);">已知已提交的最高的日志条目的索引（初始值为0，单调递增）</font> |
| <font style="color:rgb(31, 35, 40);">lastApplied</font> | <font style="color:rgb(31, 35, 40);">已经被应用到状态机的最高的日志条目的索引（初始值为0，单调递增）</font> |


<font style="color:rgb(31, 35, 40);">领导人（服务器）上的易失性状态 (选举后已经重新初始化)</font>

| <font style="color:rgb(31, 35, 40);">参数</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">nextIndex[]</font> | <font style="color:rgb(31, 35, 40);">对于每一台服务器，发送到该服务器的下一个日志条目的索引（初始值为领导人最后的日志条目的索引+1）</font> |
| <font style="color:rgb(31, 35, 40);">matchIndex[]</font> | <font style="color:rgb(31, 35, 40);">对于每一台服务器，已知的已经复制到该服务器的最高日志条目的索引（初始值为0，单调递增）</font> |


**<font style="color:rgb(31, 35, 40);">追加条目（AppendEntries）RPC</font>**<font style="color:rgb(31, 35, 40);">：</font>

<font style="color:rgb(31, 35, 40);">由领导人调用，用于日志条目的复制，同时也被当做心跳使用</font>

| <font style="color:rgb(31, 35, 40);">参数</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">term</font> | <font style="color:rgb(31, 35, 40);">领导人的任期</font> |
| <font style="color:rgb(31, 35, 40);">leaderId</font> | <font style="color:rgb(31, 35, 40);">领导人 ID 因此跟随者可以对客户端进行重定向（译者注：跟随者根据领导人 ID 把客户端的请求重定向到领导人，比如有时客户端把请求发给了跟随者而不是领导人）</font> |
| <font style="color:rgb(31, 35, 40);">prevLogIndex</font> | <font style="color:rgb(31, 35, 40);">紧邻新日志条目之前的那个日志条目的索引</font> |
| <font style="color:rgb(31, 35, 40);">prevLogTerm</font> | <font style="color:rgb(31, 35, 40);">紧邻新日志条目之前的那个日志条目的任期</font> |
| <font style="color:rgb(31, 35, 40);">entries[]</font> | <font style="color:rgb(31, 35, 40);">需要被保存的日志条目（被当做心跳使用时，则日志条目内容为空；为了提高效率可能一次性发送多个）</font> |
| <font style="color:rgb(31, 35, 40);">leaderCommit</font> | <font style="color:rgb(31, 35, 40);">领导人的已知已提交的最高的日志条目的索引</font> |


| <font style="color:rgb(31, 35, 40);">返回值</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">term</font> | <font style="color:rgb(31, 35, 40);">当前任期，对于领导人而言 它会更新自己的任期</font> |
| <font style="color:rgb(31, 35, 40);">success</font> | <font style="color:rgb(31, 35, 40);">如果跟随者所含有的条目和 prevLogIndex 以及 prevLogTerm 匹配上了，则为 true</font> |


<font style="color:rgb(31, 35, 40);">接收者的实现：</font>

1. <font style="color:rgb(31, 35, 40);">返回假 如果领导人的任期小于接收者的当前任期（译者注：这里的接收者是指跟随者或者候选人）（5.1 节）</font>
2. <font style="color:rgb(31, 35, 40);">返回假 如果接收者日志中没有包含这样一个条目 即该条目的任期在 prevLogIndex 上能和 prevLogTerm 匹配上 （译者注：在接收者日志中 如果能找到一个和 prevLogIndex 以及 prevLogTerm 一样的索引和任期的日志条目 则继续执行下面的步骤 否则返回假）（5.3 节）</font>
3. <font style="color:rgb(31, 35, 40);">如果一个已经存在的条目和新条目（译者注：即刚刚接收到的日志条目）发生了冲突（因为索引相同，任期不同），那么就删除这个已经存在的条目以及它之后的所有条目 （5.3 节）</font>
4. <font style="color:rgb(31, 35, 40);">追加日志中尚未存在的任何新条目</font>
5. <font style="color:rgb(31, 35, 40);">如果领导人的已知已提交的最高日志条目的索引大于接收者的已知已提交最高日志条目的索引（</font>`<font style="color:rgb(31, 35, 40);">leaderCommit > commitIndex</font>`<font style="color:rgb(31, 35, 40);">），则把接收者的已知已经提交的最高的日志条目的索引commitIndex 重置为 领导人的已知已经提交的最高的日志条目的索引 leaderCommit 或者是 上一个新条目的索引 取两者的最小值</font>

**<font style="color:rgb(31, 35, 40);">请求投票（RequestVote）RPC</font>**<font style="color:rgb(31, 35, 40);">：</font>

<font style="color:rgb(31, 35, 40);">由候选人负责调用用来征集选票（5.2 节）</font>

| <font style="color:rgb(31, 35, 40);">参数</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">term</font> | <font style="color:rgb(31, 35, 40);">候选人的任期号</font> |
| <font style="color:rgb(31, 35, 40);">candidateId</font> | <font style="color:rgb(31, 35, 40);">请求选票的候选人的 ID</font> |
| <font style="color:rgb(31, 35, 40);">lastLogIndex</font> | <font style="color:rgb(31, 35, 40);">候选人的最后日志条目的索引值</font> |
| <font style="color:rgb(31, 35, 40);">lastLogTerm</font> | <font style="color:rgb(31, 35, 40);">候选人最后日志条目的任期号</font> |


| <font style="color:rgb(31, 35, 40);">返回值</font> | <font style="color:rgb(31, 35, 40);">解释</font> |
| --- | --- |
| <font style="color:rgb(31, 35, 40);">term</font> | <font style="color:rgb(31, 35, 40);">当前任期号，以便于候选人去更新自己的任期号</font> |
| <font style="color:rgb(31, 35, 40);">voteGranted</font> | <font style="color:rgb(31, 35, 40);">候选人赢得了此张选票时为真</font> |


<font style="color:rgb(31, 35, 40);">接收者实现：</font>

1. <font style="color:rgb(31, 35, 40);">如果</font>`<font style="color:rgb(31, 35, 40);">term < currentTerm</font>`<font style="color:rgb(31, 35, 40);">返回 false （5.2 节）</font>
2. <font style="color:rgb(31, 35, 40);">如果 votedFor 为空或者为 candidateId，并且候选人的日志至少和自己一样新，那么就投票给他（5.2 节，5.4 节）</font>

**<font style="color:rgb(31, 35, 40);">所有服务器需遵守的规则</font>**<font style="color:rgb(31, 35, 40);">：</font>

<font style="color:rgb(31, 35, 40);">所有服务器：</font>

+ <font style="color:rgb(31, 35, 40);">如果</font>`<font style="color:rgb(31, 35, 40);">commitIndex > lastApplied</font>`<font style="color:rgb(31, 35, 40);">，则 lastApplied 递增，并将</font>`<font style="color:rgb(31, 35, 40);">log[lastApplied]</font>`<font style="color:rgb(31, 35, 40);">应用到状态机中（5.3 节）</font>
+ <font style="color:rgb(31, 35, 40);">如果接收到的 RPC 请求或响应中，任期号</font>`<font style="color:rgb(31, 35, 40);">T > currentTerm</font>`<font style="color:rgb(31, 35, 40);">，则令</font><font style="color:rgb(31, 35, 40);"> </font>`<font style="color:rgb(31, 35, 40);">currentTerm = T</font>`<font style="color:rgb(31, 35, 40);">，并切换为跟随者状态（5.1 节）</font>

<font style="color:rgb(31, 35, 40);">跟随者（5.2 节）：</font>

+ <font style="color:rgb(31, 35, 40);">响应来自候选人和领导人的请求</font>
+ <font style="color:rgb(31, 35, 40);">如果在超过选举超时时间的情况之前没有收到</font>**<font style="color:rgb(31, 35, 40);">当前领导人</font>**<font style="color:rgb(31, 35, 40);">（即该领导人的任期需与这个跟随者的当前任期相同）的心跳/附加日志，或者是给某个候选人投了票，就自己变成候选人</font>

<font style="color:rgb(31, 35, 40);">候选人（5.2 节）：</font>

+ <font style="color:rgb(31, 35, 40);">在转变成候选人后就立即开始选举过程</font>
    - <font style="color:rgb(31, 35, 40);">自增当前的任期号（currentTerm）</font>
    - <font style="color:rgb(31, 35, 40);">给自己投票</font>
    - <font style="color:rgb(31, 35, 40);">重置选举超时计时器</font>
    - <font style="color:rgb(31, 35, 40);">发送请求投票的 RPC 给其他所有服务器</font>
+ <font style="color:rgb(31, 35, 40);">如果接收到大多数服务器的选票，那么就变成领导人</font>
+ <font style="color:rgb(31, 35, 40);">如果接收到来自新的领导人的附加日志（AppendEntries）RPC，则转变成跟随者</font>
+ <font style="color:rgb(31, 35, 40);">如果选举过程超时，则再次发起一轮选举</font>

<font style="color:rgb(31, 35, 40);">领导人：</font>

+ <font style="color:rgb(31, 35, 40);">一旦成为领导人：发送空的附加日志（AppendEntries）RPC（心跳）给其他所有的服务器；在一定的空余时间之后不停的重复发送，以防止跟随者超时（5.2 节）</font>
+ <font style="color:rgb(31, 35, 40);">如果接收到来自客户端的请求：附加条目到本地日志中，在条目被应用到状态机后响应客户端（5.3 节）</font>
+ <font style="color:rgb(31, 35, 40);">如果对于一个跟随者，最后日志条目的索引值大于等于 nextIndex（</font>`<font style="color:rgb(31, 35, 40);">lastLogIndex ≥ nextIndex</font>`<font style="color:rgb(31, 35, 40);">），则发送从 nextIndex 开始的所有日志条目：</font>
    - <font style="color:rgb(31, 35, 40);">如果成功：更新相应跟随者的 nextIndex 和 matchIndex</font>
    - <font style="color:rgb(31, 35, 40);">如果因为日志不一致而失败，则 nextIndex 递减并重试</font>
+ <font style="color:rgb(31, 35, 40);">假设存在 N 满足</font>`<font style="color:rgb(31, 35, 40);">N > commitIndex</font>`<font style="color:rgb(31, 35, 40);">，使得大多数的 </font>`<font style="color:rgb(31, 35, 40);">matchIndex[i] ≥ N</font>`<font style="color:rgb(31, 35, 40);">以及</font>`<font style="color:rgb(31, 35, 40);">log[N].term == currentTerm</font>`<font style="color:rgb(31, 35, 40);"> 成立，则令 </font>`<font style="color:rgb(31, 35, 40);">commitIndex = N</font>`<font style="color:rgb(31, 35, 40);">（5.3 和 5.4 节）</font>

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/38e97a329f47def94b55e5a4894cd691.png?token=AGECE2IYHPGXU7XZTQT4JRLG72WXQ)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5e7c44946eae1da3450b61f4752b503e.png?token=AGECE2ISUDT3K3EQVP767TDG72WXW)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/e11ea13e561192f7266b29785ae82484.png?token=AGECE2MS7753SYBTNRRLGBLG72WX2)

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/573a8367663d1c7c08e43957a414ec90.png?token=AGECE2IUBCYOFNTNMXJB4DDG72WX6)

## raft基础
三种状态

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/e6d489606d87b7539100f42a9bb37129.png?token=AGECE2ILMYGCVIVEN6DXYW3G72WYC)





:::info
三个函数，实现

:::



## raft选举流程
1. follower发现心跳超时，开始发起选举
2. 首先需要把自己转换到candidate状态，然后自己的term进行加1
3. 之后发起rpc请求，来进行记录票数
4. 选举成功，他就程维leader，开始发送心跳请求给所有的人



### PartA算法实现
#### 实现基础的数据结构
<font style="background-color:#FBDE28;">定义角色，三</font>种角色follower，leader，candidate

```go
type Role string

const (
    Follower  Role = "follower"
    Candidate Role = "candidate"
    Leader    Role = "leader"
)
```

设置超时的时间ttl

```go
// setting the const of lower bound and upper bound of election time out
const (
    electionTimeoutMin time.Duration = 250 * time.Millisecond
    electionTimeOutMax time.Duration = 400 * time.Millisecond
    // the interval of the log replication
    replicateInterval time.Duration = 200 * time.Millisecond
)
```

设置关于超时的函数，是否超时，重新开始时钟

```go
// reset the election time out
func (rf *Raft) resetElectionTimeLocked() {
    // obtain a random election time out
    // plus the current time
    rf.electionStart = time.Now()
    randRange := int64(electionTimeOutMax - electionTimeoutMin)
    rf.electionTimeOut = electionTimeoutMin + time.Duration(rand.Int63()%randRange)
}

// check the election time out
func (rf *Raft) isElectionTimeOutLocked() bool {
    // just check the election time out is out of time
    return time.Since(rf.electionStart) > rf.electionTimeOut
}
```



<font style="color:#DF2A3F;">定义raft的数据结构</font>，参考图

![](https://cdn.nlark.com/yuque/0/2024/png/26023389/1721828255884-02148d65-0614-4a55-9b69-a32257fb49c7.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1098%2Climit_0)

1. currentterm当前的任期
2. votedfor选举的是谁，使用-1代表没有进行选举
3. 超时时间还有开始时间



```go
type Raft struct {
    mu        sync.Mutex          // Lock to protect shared access to this peer's state
    peers     []*labrpc.ClientEnd // RPC end points of all peers
    persister *Persister          // Object to hold this peer's persisted state
    me        int                 // this peer's index into peers[]
    dead      int32               // set by Kill()
// 添加的数据结构，定义什么时候开始选举的，还有超时时间
    role            Role
    currentTerm     int
    votedFor        int //vote who (-1 present null)
    electionStart   time.Time
    electionTimeOut time.Duration
    // Your data here (PartA, PartB, PartC).
    // add yourself struct
    // Look at the paper's Figure 2 for a description of what
    // state a Raft server must maintain.

}
```



实现become函数

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/710d67fdd66d08b9b394973ce1947257.png?token=AGECE2JTVRXGAGY6QSJ32OLG72WYQ)

变成follower

1. 重设时钟
2. 更新term，设置角色为follower，并且设置为没有进行选举

```go
func (rf *Raft) becomeFollowerLocked(term int) {
    // compare term ,if term is lower than rf.term ,do not change
    if rf.currentTerm > term {
        // add log
        LOG(rf.me, rf.currentTerm, DError, "Can't become Follower, lower term: T%d", term)
        return
    }
    // log
    LOG(rf.me, rf.currentTerm, DLog, "%s->Follower, For T%v->T%v", rf.role, rf.currentTerm, term)
    rf.role = Follower
    if rf.currentTerm < term {
        rf.votedFor = -1
    }
    rf.currentTerm = term
    // all term

}
```



变成candidate

1. 自己是leader就不用变了（变成candidate就是为了变成leader）
2. 设置角色为candidate，更新votedfor是自己，设置自己currentterm进行加1

```go
func (rf *Raft) becomeCandidateLocked() {
    // compare term ,if term is lower than rf.term ,do not change
    if rf.role == Leader {
        // add log
        LOG(rf.me, rf.currentTerm, DError, "Leader can't become Candidate")
        return
    }
    // log
    LOG(rf.me, rf.currentTerm, DVote, "%s->Candidate, For T%d", rf.role, rf.currentTerm+1)
    rf.role = Candidate
    // 选举人的任期+1,并且投票自己
    rf.currentTerm++
    rf.votedFor = rf.me
    // me 代表序号
    // all term

}
```



<font style="background-color:#FBDE28;">变成leader</font>

1. 只有candidate才可以变成leader

```go
func (rf *Raft) becomeLeaderLocked() {

    if rf.role != Candidate {
        LOG(rf.me, rf.currentTerm, DError, "Only Candidate can become Leader")
        return
        //
    }
    LOG(rf.me, rf.currentTerm, DLeader, "Become Leader in T%d", rf.currentTerm)

    rf.role = Leader
}
```





开始进行初始化数据结构

```go
func Make(peers []*labrpc.ClientEnd, me int,
    persister *Persister, applyCh chan ApplyMsg) *Raft {
    rf := &Raft{}
    rf.peers = peers
    rf.persister = persister
    rf.me = me

    // Your initialization code here (PartA, PartB, PartC).
    /*
        init the filed that you have added
    */

    rf.role = Follower
    rf.currentTerm = 0
    rf.votedFor = -1
    // initialize from state persisted before a crash
    rf.readPersist(persister.ReadRaftState())

    // start ticker goroutine to start elections
    go rf.electionTicker()

    return rf
}
```



初始化之后就进入到electiontricker代码。



#### 实现选举electiontricker
```go
// Your code here (PartA)
        // Check if a leader election should be started.
```

需要进行检查是否需要开始，开始选举的条件就是没有超时，而且，自己还不是leader

因为是并发操作，需要使用mutex进行上锁



```go
func (rf *Raft) electionTicker() {
    for rf.killed() == false {

        // Your code here (PartA)
        // Check if a leader election should be started.

        // here only you are not leader ,you can start election,at the same time
        //  you should check the election time out
        rf.mu.Lock()
        if rf.role != Leader && rf.isElectionTimeOutLocked() {
            rf.becomeCandidateLocked()
            // according to the term ,send request vote to all the peers
            // only you term is higher than the other ,you can get the vote
            go rf.startElection(rf.currentTerm)
        }
        rf.mu.Unlock()

        // pause for a random amount of time between 50 and 350
        // milliseconds.
        ms := 50 + (rand.Int63() % 300)
        time.Sleep(time.Duration(ms) * time.Millisecond)
    }
}
```

主要流程

1. 变成candidate
2. 发起选举，实现startelection



startelection需要进行rpc调用，来进行计算得票数，一个发起rpc，还有一个需要接受rpc来进行回复

startelection需要计算得票数，超过一半，就是要进行变为leader

1. 对于所有的peers队列，发起一次rpc请求，如果是自己，直接跳过，并把票数+1，
2. 不是就要开始构建rpc的发送结<font style="background-color:#FBDE28;">构体还有接受结构体（这个参考论文实现）</font>
3. <font style="background-color:#FBDE28;">然后开始进行调用rpc来进行计算得票数</font>，主要操作就是来实现一个send函数，和一个call函数
4. 根据是否授予来加票，超过一半，直接变成leader



结构体构建rpc的send还有reply

参考下面的图，任期还有leaderid

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/45703376fd691cec8cfe388d203c81ad.png?token=AGECE2JHLCZKDYTYGKJBMSDG72WYU)

```go
type AppendEntriesArgs struct {
    Term     int
    LeaderID int
}

// rpc reply
type AppendEntriesReply struct {
    Term    int
    Success bool
}
```



开始发送rpc调用，直接使用已经有的函数

```go
func (rf *Raft) sendRequestVote(server int, args *RequestVoteArgs, reply *RequestVoteReply) bool {
    ok := rf.peers[server].Call("Raft.RequestVote", args, reply)
    return ok
}
```

这里是rpc发送端，因此也需要实现rpc接受端，定位到call的requestvote这个函数



```go
// example RequestVote RPC handler.
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
    // Your code here (PartA, PartB).
```

接收端的操作

1. 首先来进行加锁
2. 根据接收到的参数来进行设置reply
3. 之后开始来进行比较term，如果args的term小于当前我自己的term，那么直接失败，不能发起选举
4. 如果是大于，那么我就变成follower
5. 之后查看当前的是否已经选举别人，只要是-1，就说明没有支持其他人
6. 最后设置返回体，重设时钟



```go
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
    // Your code here (PartA, PartB).
    // this function is to handle the RequestVote RPC,when receive rpc
    // what should server do
    rf.mu.Lock()
    defer rf.mu.Unlock()
    reply.Term = rf.currentTerm
    reply.VoteGranted = false
    // if the term is lower than the current term ,reject the vote
    if args.Term < rf.currentTerm {
        LOG(rf.me, rf.currentTerm, DVote, "<- S%d, Reject voted, Higher term, T%d>T%d", args.CandidateID, rf.currentTerm, args.Term)
        return

    }
    // if the term is higher than the current term ,become follower
    if args.Term > rf.currentTerm {
        rf.becomeFollowerLocked(args.Term)
    }
    // if the server has voted for other ,reject the vote
    if rf.votedFor != -1 {
        LOG(rf.me, rf.currentTerm, DVote, "<- S%d, Reject voted, Already voted to S%d", args.CandidateID, rf.votedFor)
        return

    }
    // all pass ,setting reply
    reply.VoteGranted = true
    rf.votedFor = args.CandidateID
    // reset clock
    rf.resetElectionTimeLocked()
    LOG(rf.me, rf.currentTerm, DVote, "<- S%d, Vote granted", args.CandidateID)
}
```



有了rpc之后就是要实现上面的rpc调用选举，之前的思路来发送请求

```go
// build RPC call for each peer ,via peer &args
    askVoteFromPeer := func(peer int, args *RequestVoteArgs) {
        // this is client call
        reply := &RequestVoteReply{}
        // above code ,given  a example to call rpc,just pass arg and reply
        // return ok
        ok := rf.sendRequestVote(peer, args, reply)
        rf.mu.Lock()
        defer rf.mu.Unlock()
        // 多线程竞争
        if !ok {
            // rpc failed
            LOG(rf.me, rf.currentTerm, DDebug, "-> S%d, Ask vote, Lost or error", peer)
            return
        }
        LOG(rf.me, rf.currentTerm, DDebug, "-> S%d, AskVote Reply=", peer)

        // if currentTerm is lower than peer,become follower
        if reply.Term > rf.currentTerm {
            rf.becomeFollowerLocked(reply.Term)
            return
        }
        // double check current role is Candidate && term is equal
        if rf.contextLostLocked(Candidate, term) {
            LOG(rf.me, rf.currentTerm, DVote, "-> S%d, Lost context, abort RequestVoteReply", peer)
            return
        }
        // if vote granted ,vote++
        if reply.VoteGranted {
            vote++
            // when half of the peer vote ,become leader
            if vote > len(rf.peers)/2 {
                rf.becomeLeaderLocked()
                // start the leader work,send heartbeat & log replication
                go rf.replicationTicker(term)
            }
        }
    }
```



```go
func (rf *Raft) contextLostLocked(role Role, term int) bool {
    return rf.role != role || rf.currentTerm != term

}
```

:::info
主要到两个if提前判断出局，因为有可能当前的返回可能大于自己的term，自己就要变成follower，或者是自己的角色已经变了，因为是在多线程条件下，任期也变了，所以直接返回了

:::



#### 实现心跳逻辑
主要还是参考之前的选举逻辑，主要的目的就是确保各个peer是否还是存在于活跃期间，如果不是就需要进行返回false，rpc的回调函数比较简单就是接受到之后，来进行判断term是不是leader的大，，是的话就需要变成follower，peer，自己重新设置时钟

```go
func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
    rf.mu.Lock()
    defer rf.mu.Unlock()

    reply.Term = rf.currentTerm
    reply.Success = false
    // align the term
    if args.Term < rf.currentTerm {
        LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject log", args.LeaderId)
        return
    }
    if args.Term >= rf.currentTerm {
        rf.becomeFollowerLocked(args.Term)
    }

    // reset the timer
    rf.resetElectionTimerLocked()
    reply.Success = true
}
```



剩下就是接受到结果后leader的行为，需要进行查看自己还是不是leader，因为可能一轮结束后，重新发了选举，自己已经被变成follower因此需要来进行校验context loss

1. 接收到返回结果
2. 查看是否成功，失败就返回
3. 成功来比较term，如果小于就是需要变成follower

```go
replicateToPeer := func(peer int, args *AppendEntriesArgs) {
    reply := &AppendEntriesReply{}
    ok := rf.sendAppendEntries(peer, args, reply)

    rf.mu.Lock()
    defer rf.mu.Unlock()
    if !ok {
        LOG(rf.me, rf.currentTerm, DLog, "-> S%d, Lost or crashed", peer)
        return
    }
    // align the term
    if reply.Term > rf.currentTerm {
        rf.becomeFollowerLocked(reply.Term)
        return
    }
}
```

```go
func (rf *Raft) startReplication(term int) bool {
    replicateToPeer := func(peer int, args *AppendEntriesArgs) {
        // send heartbeat RPC and handle the reply
    }

    rf.mu.Lock()
    defer rf.mu.Unlock()
    if rf.contextLostLocked(Leader, term) {
        LOG(rf.me, rf.currentTerm, DLeader, "Leader[T%d] -> %s[T%d]", term, rf.role, rf.currentTerm)
        return false
    }

    for peer := 0; peer < len(rf.peers); peer++ {
        if peer == rf.me {
            continue
        }

        args := &AppendEntriesArgs{
            Term:     term,
            LeaderId: rf.me,
        }

        go replicateToPeer(peer, args)
    }

    return true
}
```





整体的时钟计时器代开，参考之前的选举，也是for，然后进行校验

```go

func (rf *Raft) replicationTicker(term int) {
        for !rf.killed() {
                ok := rf.startReplication(term)
                if !ok {
                        return
                }

                time.Sleep(replicateInterval)
        }
}

```

启动流程是需要选举成功之后开始打开当前这个的定时器

```go
if rf.contextLostLocked(Candidate, term) {
    LOG(rf.me, rf.currentTerm, DVote, "-> S%d, Lost context, abort RequestVoteReply", peer)
    return
}

// count the votes
if reply.VoteGranted {
    votes++
    if votes > len(rf.peers)/2 {
        rf.becomeLeaderLocked()
        go rf.replicationTicker(term)
    }
}
```

## raft复制流程-PartB
![画板](https://raw.githubusercontent.com/weijia99/sync2hexo/master/c35350b47483bb91f2d6e15dbc3f6481.jpeg?token=AGECE2MHPP4C6PL2BFYPCFTG72WYY)



> <font style="color:rgb(31, 35, 40);">一旦一个领导人被选举出来，他就开始为客户端提供服务。客户端的每一个请求都包含一条被复制状态机执行的指令。领导人把这条指令作为一条新的日志条目附加到日志中去，然后并行地发起附加条目 RPCs 给其他的服务器，让他们复制这条日志条目。当这条日志条目被安全地复制（下面会介绍），领导人会应用这条日志条目到它的状态机中然后把执行的结果返回给客户端。如果跟随者崩溃或者运行缓慢，再或者网络丢包，领导人会不断的重复尝试附加日志条目 RPCs （尽管已经回复了客户端）直到所有的跟随者都最终存储了所有的日志条目。</font>
>

:::info
总体流程分为下面三个步骤1.进行复制log entries 给每一个peer，因此需要记录已经匹配了的每一个peer的nextmatch数组2.进行发送rpc请求参考之前的选举逻辑。3.rpc返回已近成功加入的长度，选择大多数peer都已经达到的长度作为commit index，进行应用。

:::

### 3.1复制逻辑数据结构
1.首先是实现基础的数据结构，根据上文的内容，我们需要nextindex作为记录匹配点的记录，同时这个matchindex是已经进行应用log的数组，数组大小都是peer的长度。同时还是需要log来进行记录每一条命令来运用

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/fcf8b92bd105568ba1aba681c6b6daa2.png?token=AGECE2IPME3VKCFWUAZMKB3G72WY6)

跟新raft结构体

```go
type LogEntry struct {
        Term         int
        CommandValid bool
        Command      interface{}
}

// log in Peer's local
log         []LogEntry

// only used when it is Leader,
// log view for each peer
nextIndex  []int
matchIndex []int
```





接下来就是进行初始化这些数组在raft里面的make函数进行更新



```go
func Make(peers []*labrpc.ClientEnd, me int,
        persister *Persister, applyCh chan ApplyMsg) *Raft {
        // ......
        rf.log = append(rf.log, LogEntry{})

        rf.matchIndex = make([]int, len(rf.peers))
        rf.nextIndex = make([]int, len(rf.peers))

        // ......
}
```





### 3.2实现发送rpc请求
> + <font style="color:rgb(31, 35, 40);">一旦成为领导人：发送空的附加日志（AppendEntries）RPC（心跳）给其他所有的服务器；在一定的空余时间之后不停的重复发送，以防止跟随者超时（5.2 节）</font>
> + <font style="color:rgb(31, 35, 40);">如果接收到来自客户端的请求：附加条目到本地日志中，在条目被应用到状态机后响应客户端（5.3 节）</font>
> + <font style="color:rgb(31, 35, 40);">如果对于一个跟随者，最后日志条目的索引值大于等于 nextIndex（</font>`<font style="color:rgb(31, 35, 40);">lastLogIndex ≥ nextIndex</font>`<font style="color:rgb(31, 35, 40);">），则发送从 nextIndex 开始的所有日志条目：</font>
>     - <font style="color:rgb(31, 35, 40);">如果成功：更新相应跟随者的 nextIndex 和 matchIndex</font>
>     - <font style="color:rgb(31, 35, 40);">如果因为日志不一致而失败，则 nextIndex 递减并重试</font>
> + <font style="color:rgb(31, 35, 40);">假设存在 N 满足</font>`<font style="color:rgb(31, 35, 40);">N > commitIndex</font>`<font style="color:rgb(31, 35, 40);">，使得大多数的 </font>`<font style="color:rgb(31, 35, 40);">matchIndex[i] ≥ N</font>`<font style="color:rgb(31, 35, 40);">以及</font>`<font style="color:rgb(31, 35, 40);">log[N].term == currentTerm</font>`<font style="color:rgb(31, 35, 40);"> 成立，则令 </font>`<font style="color:rgb(31, 35, 40);">commitIndex = N</font>`<font style="color:rgb(31, 35, 40);">（5.3 和 5.4 节）</font>
>

<font style="color:rgb(31, 35, 40);">下面就是需要实现进行发送rpc请求，需要的字段，</font>

参考下面图二的数据结构，整体流程和之前发送rpc的请求逻辑差不多，也是进行构建三个逻辑。

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/213c1733a635999e29944ada9fa31518.png?token=AGECE2LZNCTTPS2HC4GDSD3G72WZC)

数据结构设置，发送rpc的目的是为了让对方接受到自己entr，之后应用entries到各自folower的log上面，因此需要进行记录。

```go
// add log entries according to the ApplyMsg struct
type LogEntry struct {
        Term         int
        CommandValid bool
        Command      interface{}
}

// add the fields about log: 
// PrevLogIndex and PrevLogTerm is used to match the log prefix
// Entries is used to append when matched
// LeaderCommit tells the follower to update its own commitIndex
type AppendEntriesArgs struct {
        Term     int
        LeaderId int

        PrevLogIndex int
        PrevLogTerm  int
        Entries      []LogEntry
}

type AppendEntriesReply struct {
        Term    int
        Success bool
}
```

:::info
使用leaderid是用来便于调试，是哪一个是leader，剩下的结构于之前的发送心跳差不多，主要是多了之前的prevLogIndex表示之前已经匹配的log

:::



下面实现之前的操作，回调函数，发送rpc，接受收到rpc的处理。

#### 回调函数
主要处理逻辑，就是检查index事后的term于发过来的term是不是一样，一样的话就进行把entries添加到自己log里面，然后返回

1. 比较长度问题，如果index长度大于peer自己的长度就是直接false
2. 如果当前的index的term于peer的term不一样也是要返回false

```go
func (rf *Raft) AppendEntries(args *AppendEntriesArgs, reply *AppendEntriesReply) {
        rf.mu.Lock()
        defer rf.mu.Unlock()

        // For debug
        LOG(rf.me, rf.currentTerm, DDebug, "<- S%d, Receive log, Prev=[%d]T%d, Len()=%d", args.LeaderId, args.PrevLogIndex, args.PrevLogTerm, len(args.Entries))
        // replay initialized
        reply.Term = rf.currentTerm
        reply.Success = false

        // align the term
        if args.Term < rf.currentTerm {
                LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject Log, Higher term, T%d<T%d", args.LeaderId, args.Term, rf.currentTerm)
                return
        }
        if args.Term >= rf.currentTerm {
                rf.becomeFollowerLocked(args.Term)
        }

        // return failure if the previous log not matched
        if args.PrevLogIndex >= len(rf.log) {
                LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject Log, Follower log too short, Len:%d <= Prev:%d", args.LeaderId, len(rf.log), args.PrevLogIndex)
                return
        }
        if rf.log[args.PrevLogIndex].Term != args.PrevLogTerm {
                LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject Log, Prev log not match, [%d]: T%d != T%d", args.LeaderId, args.PrevLogIndex, rf.log[args.PrevLogIndex].Term, args.PrevLogTerm)
                return
        }

        // append the leader logs to local
        rf.log = append(rf.log[:args.PrevLogIndex+1], args.Entries...)
        LOG(rf.me, rf.currentTerm, DLog2, "Follower append logs: (%d, %d]", args.PrevLogIndex, args.PrevLogIndex+len(args.Entries))
        reply.Success = true

        // TODO: handle the args.LeaderCommit

        // reset the election timer, promising not start election in some interval
        rf.resetElectionTimerLocked()
}
```

新增代码如下，进行检查，然后进行append，然后查看commiedindex

:::info
  if args.PrevLogIndex >= len(rf.log) {

                LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject Log, Follower log too short, Len:%d <= Prev:%d", args.LeaderId, len(rf.log), args.PrevLogIndex)

                return

        }

        if rf.log[args.PrevLogIndex].Term != args.PrevLogTerm {

                LOG(rf.me, rf.currentTerm, DLog2, "<- S%d, Reject Log, Prev log not match, [%d]: T%d != T%d", args.LeaderId, args.PrevLogIndex, rf.log[args.PrevLogIndex].Term, args.PrevLogTerm)

                return

        }



        // append the leader logs to local

        rf.log = append(rf.log[:args.PrevLogIndex+1], args.Entries...)

        LOG(rf.me, rf.currentTerm, DLog2, "Follower append logs: (%d, %d]", args.PrevLogIndex, args.PrevLogIndex+len(args.Entries))

        reply.Success = true



        // TODO: handle the args.LeaderCommit

:::

#### 实现发送逻辑
主要的操作就是构建rpc发送的请求结构体，然后处理接受之后的rc返回

1. 构建rpc请求，preindx，是nextindex的前一个，表示已经处理完成的term
2. 对于rpc进行同步日志失败的，需要进行回车term，回到最开始的term的index然后更新nextindex，在发送rpc就是上一个term的最后一次，开一进行比较term是不是一样了（相当于term减少一发送rpc）



首先是构造rpc请求

```go
 for peer := 0; peer < len(rf.peers); peer++ {
                if peer == rf.me {
                        // Don't forget to update Leader's matchIndex
                        rf.matchIndex[peer] = len(rf.log) - 1
                        rf.nextIndex[peer] = len(rf.log)
                        continue
                }

                prevIdx := rf.nextIndex[peer] - 1
                prevTerm := rf.log[prevIdx].Term
                args := &AppendEntriesArgs{
                        Term:         rf.currentTerm,
                        LeaderId:     rf.me,
                        PrevLogIndex: prevIdx,
                        PrevLogTerm:  prevTerm,
                        Entries:      rf.log[prevIdx+1:],
                        LeaderCommit: rf.commitIndex,
                }
                LOG(rf.me, rf.currentTerm, DDebug, "-> S%d, Send log, Prev=[%d]T%d, Len()=%d", peer, args.PrevLogIndex, args.PrevLogTerm, len(args.Entries))
                go replicateToPeer(peer, args)
        }
```

1. 选择之前preindex
2. 发送的entry则是从prev之后的



<font style="background-color:#FBDE28;">接受后的处理代码</font>

1. 检查是否成功
2. 不成功说明当前的index不适合，需要我们进行回退上一个的term的结尾的index
3. 成功就是需要进行更新nextindex



```go
replicateToPeer := func(peer int, args *AppendEntriesArgs) {
                reply := &AppendEntriesReply{}
                ok := rf.sendAppendEntries(peer, args, reply)

                rf.mu.Lock()
                defer rf.mu.Unlock()
                if !ok {
                        LOG(rf.me, rf.currentTerm, DLog, "-> S%d, Lost or crashed", peer)
                        return
                }

                // align the term
                if reply.Term > rf.currentTerm {
                        rf.becomeFollowerLocked(reply.Term)
                        return
                }

                // probe the lower index if the prev log not matched
                if !reply.Success {
                        idx := rf.nextIndex[peer] - 1
                        term := rf.log[idx].Term
                        for idx > 0 && rf.log[idx].Term == term {
                                idx--
                        }
                        rf.nextIndex[peer] = idx + 1
                        LOG(rf.me, rf.currentTerm, DLog, "Log not matched in %d, Update next=%d", args.PrevLogIndex, rf.nextIndex[peer])
                        return
                }

                // update the match/next index if log appended successfully
                rf.matchIndex[peer] = args.PrevLogIndex + len(args.Entries)
                rf.nextIndex[peer] = rf.matchIndex[peer] + 1

                // TODO: need compute the new commitIndex here, 
                // but we leave it to the other chapter
        }
```



<font style="background-color:#FBDE28;">整体的代码</font>

```go
func (rf *Raft) startReplication(term int) bool {
        replicateToPeer := func(peer int, args *AppendEntriesArgs) {
                reply := &AppendEntriesReply{}
                ok := rf.sendAppendEntries(peer, args, reply)

                rf.mu.Lock()
                defer rf.mu.Unlock()
                if !ok {
                        LOG(rf.me, rf.currentTerm, DLog, "-> S%d, Lost or crashed", peer)
                        return
                }

                // align the term
                if reply.Term > rf.currentTerm {
                        rf.becomeFollowerLocked(reply.Term)
                        return
                }

                // probe the lower index if the prev log not matched
                if !reply.Success {
                        idx := rf.nextIndex[peer] - 1
                        term := rf.log[idx].Term
                        for idx > 0 && rf.log[idx].Term == term {
                                idx--
                        }
                        rf.nextIndex[peer] = idx + 1
                        LOG(rf.me, rf.currentTerm, DLog, "Log not matched in %d, Update next=%d", args.PrevLogIndex, rf.nextIndex[peer])
                        return
                }

                // update the match/next index if log appended successfully
                rf.matchIndex[peer] = args.PrevLogIndex + len(args.Entries)
                rf.nextIndex[peer] = rf.matchIndex[peer] + 1

                // TODO: need compute the new commitIndex here, 
                // but we leave it to the other chapter
        }

        rf.mu.Lock()
        defer rf.mu.Unlock()

        if rf.contextLostLocked(Leader, term) {
                LOG(rf.me, rf.currentTerm, DLog, "Lost Leader[%d] to %s[T%d]", term, rf.role, rf.currentTerm)
                return false
        }

        for peer := 0; peer < len(rf.peers); peer++ {
                if peer == rf.me {
                        // Don't forget to update Leader's matchIndex
                        rf.matchIndex[peer] = len(rf.log) - 1
                        rf.nextIndex[peer] = len(rf.log)
                        continue
                }

                prevIdx := rf.nextIndex[peer] - 1
                prevTerm := rf.log[prevIdx].Term
                args := &AppendEntriesArgs{
                        Term:         rf.currentTerm,
                        LeaderId:     rf.me,
                        PrevLogIndex: prevIdx,
                        PrevLogTerm:  prevTerm,
                        Entries:      rf.log[prevIdx+1:],
                        LeaderCommit: rf.commitIndex,
                }
                LOG(rf.me, rf.currentTerm, DDebug, "-> S%d, Send log, Prev=[%d]T%d, Len()=%d", peer, args.PrevLogIndex, args.PrevLogTerm, len(args.Entries))
                go replicateToPeer(peer, args)
        }

        return true
}
```



初始化数组长度，只有自己变成leader才需要进行初始化next和match，

```go
func (rf *Raft) becomeLeaderLocked() {
        if rf.role != Candidate {
                LOG(rf.me, rf.currentTerm, DError, "Only Candidate can become Leader")
                return
        }

        LOG(rf.me, rf.currentTerm, DLeader, "Become Leader in T%d", rf.currentTerm)
        rf.role = Leader
        for peer := 0; peer < len(rf.peers); peer++ {
                rf.nextIndex[peer] = len(rf.log)
                rf.matchIndex[peer] = 0
        }
}
```





### 
### 3.3完善选举逻辑
![画板](https://raw.githubusercontent.com/weijia99/sync2hexo/master/416e8679c5d88a6e53f46603552fb0d8.jpeg?token=AGECE2OAISYBFUZJHJU3O43G72WZE)

添加了log之后选举的逻辑就变成要看最后一个index的term，是不是最后的term大于对方的，

[7.2 选举约束（Election Restriction） | MIT6.824 (gitbook.io)](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/lecture-07-raft2/7.2-xuan-ju-yue-shu-election-restriction)

> 1.   
候选人最后一条Log条目的任期号**大于**本地最后一条Log条目的任期号；
> 2. 或者，候选人最后一条Log条目的任期号**等于**本地最后一条Log条目的任期号，且候选人的Log记录长度**大于等于**本地Log记录的长度
>

1. 比较最后的term
2. term相同，就比较两个的index

```go
func (rf *Raft) isMoreUpToDateLocked(candidateIndex, candidateTerm int) bool {
        l := len(rf.log)
        lastTerm, lastIndex := rf.log[l-1].Term, l-1
        LOG(rf.me, rf.currentTerm, DVote, "Compare last log, Me: [%d]T%d, Candidate: [%d]T%d", lastIndex, lastTerm, candidateIndex, candidateTerm)

        if lastTerm != candidateTerm {
                return lastTerm > candidateTerm
        }
        return lastIndex > candidateIndex
}
```

因为要进行比较，所以需要增加字段在发送rpc的选举时候

```go
type RequestVoteArgs struct {
        // Your data here (PartA, PartB).
        Term         int
        CandidateId  int
        LastLogIndex int
        LastLogTerm  int
}
```

#### 更新rpc发送的字段
```go
for peer := 0; peer < len(rf.peers); peer++ {
                if peer == rf.me {
                        votes++
                        continue
                }

                args := &RequestVoteArgs{
                        Term:         rf.currentTerm,
                        CandidateId:  rf.me,
                        LastLogIndex: l-1,
                        LastLogTerm:  rf.log[l-1].Term,
                }
```

#### 更新回调函数
```go

        // check log, only grante vote when the candidates have more up-to-date log
        if rf.isMoreUpToDateLocked(args.LastLogIndex，args.LastLogTerm) {
                LOG(rf.me, rf.currentTerm, DVote, "-> S%d, Reject Vote, S%d's log less up-to-date", args.CandidateId)
                return
        }

```



### 应用日志
raft通过复制使用同一个log来达到相同的状态，通过commitidx变大，触发更新（只有leader有一大半的收到了，那就是说明一大半follower已经有这个commit之前的日志，可以进行更新apply



![画板](https://raw.githubusercontent.com/weijia99/sync2hexo/master/ad6ec12b158d4e681714ddbd80fdd0b9.jpeg?token=AGECE2LYXYDBBAFKXGGDPHDG72WZK)



现在就是应有rpc的log数组，然后惊醒应用log，这里需要使用channel来使用

1. 首先根据比较commit与peer的commit的大小，peer小于leader，就出发更新，这里可以使用一个条件控制来打开
2. 对于leader，接收到超过一半matchindex就可以看做是可以更新
3. 更新方式使用timer来作为，主要就是运用所有的entry转变为msg，通过管道使用
4. **<font style="color:rgb(51, 51, 51);">lastApplied</font>**<font style="color:rgb(51, 51, 51);">：本 Peer 日志 apply 进度</font>

#### 更新数据结构
```go
   // commit index and last applied
        commitIndex int
        lastApplied int
        applyCond   *sync.Cond
        applyCh     chan ApplyMsg
```

```go
  
        rf.applyCh = applyCh
        rf.commitIndex = 0
        rf.lastApplied = 0
        rf.applyCond = sync.NewCond(&rf.mu)

        // initialize from state persisted before a crash
        rf.readPersist(persister.ReadRaftState())

        // start ticker goroutine to start elections
        go rf.electionTicker()
        go rf.applyTicker()
```

applyticker一直在更新

1. 从applied+1开始获取数组，放入到entries
2. 然后构建msg，使用channel使用
3. 更新applied长度



```go
func (rf *Raft) applyTicker() {
        for !rf.killed() {
                rf.mu.Lock()
                rf.applyCond.Wait()

                entries := make([]LogEntry, 0)
                // should start from rf.lastApplied+1 instead of rf.lastApplied
                for i := rf.lastApplied + 1; i <= rf.commitIndex; i++ {
                        entries = append(entries, rf.log[i])
                }
                rf.mu.Unlock()

                for i, entry := range entries {
                        rf.applyCh <- ApplyMsg{
                                CommandValid: entry.CommandValid,
                                Command:      entry.Command,
                                CommandIndex: rf.lastApplied + 1 + i,
                        }
                }

                rf.mu.Lock()
                LOG(rf.me, rf.currentTerm, DApply, "Apply log for [%d, %d]", rf.lastApplied+1, rf.lastApplied+len(entries))
                rf.lastApplied += len(entries)
                rf.mu.Unlock()
        }
}
```

#### 更新leader的applied
对于每一个rpc回调后，都有更新一次nextindex还有matchindex，我们吧matchindex排序获取中间值就是超过一半的index，如果大于自己的commitindex就是需要更新，打开条件就可以进行appliedtricker

```go

replicateToPeer := func(peer int, args *AppendEntriesArgs) {
        // ......

        // update the commmit index if log appended successfully
        rf.matchIndex[peer] = args.PrevLogIndex + len(args.Entries)
        rf.nextIndex[peer] = rf.matchIndex[peer] + 1 // important: must update
        majorityMatched := rf.getMajorityIndexLocked()
        if majorityMatched > rf.commitIndex {
                LOG(rf.me, rf.currentTerm, DApply, "Leader update the commit index %d->%d", rf.commitIndex, majorityMatched)
                rf.commitIndex = majorityMatched
                rf.applyCond.Signal()
        }
}
```

获取众数的代码

```go
func (rf *Raft) getMajorityIndexLocked() int {
        // TODO(spw): may could be avoid copying
        tmpIndexes := make([]int, len(rf.matchIndex))
        copy(tmpIndexes, rf.matchIndex)
        sort.Ints(sort.IntSlice(tmpIndexes))
        majorityIdx := (len(tmpIndexes) - 1) / 2
        LOG(rf.me, rf.currentTerm, DDebug, "Match index after sort: %v, majority[%d]=%d", tmpIndexes, majorityIdx, tmpIndexes[majorityIdx])
        return tmpIndexes[majorityIdx] // min -> max
}
```



#### 对follower更新
在进行rpc调用的时候传入了commitind，如果比peer的大，就是需要更新调用了

```go
 // update the commit index if needed and indicate the apply loop to apply
        if args.LeaderCommit > rf.commitIndex {
                LOG(rf.me, rf.currentTerm, DApply, "Follower update the commit index %d->%d", rf.commitIndex, args.LeaderCommit)
                rf.commitIndex = args.LeaderCommit
                if rf.commitIndex >= len(rf.log) {
                    rf.commitIndex = len(rf.log) - 1
                }
                rf.applyCond.Signal()
        }
```



## 持久化实现
### 4.1序列化与饭序列化
直接参照代码的实力

```go
func (rf *Raft) persistLocked() {
        w := new(bytes.Buffer)
        e := labgob.NewEncoder(w)
        e.Encode(rf.currentTerm)
        e.Encode(rf.votedFor)
        e.Encode(rf.log)
        raftstate := w.Bytes()
        // leave the second parameter nil, will use it in PartD
        rf.persister.Save(raftstate, nil) 
}

// restore previously persisted state.
func (rf *Raft) readPersist(data []byte) {
        if data == nil || len(data) < 1 {
                return
        }

        var currentTerm int
        var votedFor int
        var log []LogEntry

        r := bytes.NewBuffer(data)
        d := labgob.NewDecoder(r)
        if err := d.Decode(&currentTerm); err != nil {
                LOG(rf.me, rf.currentTerm, DPersist, "Read currentTerm error: %v", err)
                return
        }
        rf.currentTerm = currentTerm

        if err := d.Decode(&votedFor); err != nil {
                LOG(rf.me, rf.currentTerm, DPersist, "Read votedFor error: %v", err)
                return
        }
        rf.votedFor = votedFor

        if err := d.Decode(&log); err != nil {
                LOG(rf.me, rf.currentTerm, DPersist, "Read log error: %v", err)
                return
        }
        rf.log = log
        LOG(rf.me, rf.currentTerm, DPersist, "Read Persist %v", rf.stateString())
}
```

根据论文要求，持久化需要修改的currentterm，votedfor还有log【】，只要批量查找更新这几个的就进行持久化就行

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/5c64617eb09155ce51d7424d50b31b19.png?token=AGECE2PKGFMUBG7CIAAVVBDG72WZQ)



#### 
### 4.2快速回复
参考[7.3 快速恢复（Fast Backup） | MIT6.824 (gitbook.io)](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/lecture-07-raft2/7.3-hui-fu-jia-su-backup-acceleration)

大致意识就是，使用之前的方法进行回撤，需要经过一个for周期才能开始查找，浪费了时间，能不能直接一次就开始查找下一个

1. 如果pre大于peer的长度，返回的index就是peer的长度
2. 如果是在term不相等，那就找到term时候的第一条日志作为conflict index和term进行返回
3. 返回后来更新term，如果是term原因，leader也查找这个term的index，有就作为更新的起始，没有就用follower的序列作为起始来进行更新

**<font style="color:rgb(51, 51, 51);">让 Follower 给点信息</font>**<font style="color:rgb(51, 51, 51);">——告诉 Leader 自己日志大致到哪里了！</font>

<font style="color:rgb(51, 51, 51);">于是我们给 AppendEntriesReply 增加两个额外的字段，以携带一些 Follower 和 Leader 冲突日志的信息。</font>

```go
type AppendEntriesReply struct {
    Term    int
    Success bool

    ConfilictIndex int
    ConfilictTerm  int
}
```

<font style="color:rgb(51, 51, 51);">Follower 端大概算法如下：</font>

1. <font style="color:rgb(51, 51, 51);">如果 Follower 日志过短，则</font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 置空， </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictIndex = len(rf.log)</font>`<font style="color:rgb(51, 51, 51);">。</font>
2. <font style="color:rgb(51, 51, 51);">否则，将 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 设置为 Follower 在 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">Leader.PrevLogIndex</font>`<font style="color:rgb(51, 51, 51);"> 处日志的 term；</font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictIndex</font>`<font style="color:rgb(51, 51, 51);"> 设置为 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 的第一条日志。</font>

<font style="color:rgb(51, 51, 51);">第一条做法的目的在于，如果 Follower 日志过短，可以提示 Leader 迅速回退到 Follower 日志的末尾，而不用傻傻的一个个 index 或者 term 往前试探。</font>

<font style="color:rgb(51, 51, 51);">第二条的目的在于，如果 Follower 存在 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">Leader.PrevLog</font>`<font style="color:rgb(51, 51, 51);"> ，但不匹配，则将对应 term 的日志全部跳过。</font>

```go
// --- rf.AppendEntries in raft_replication.go

// return failure if prevLog not matched
if args.PrevLogIndex >= len(rf.log) {
    reply.ConfilictIndex = len(rf.log)
    reply.ConfilictTerm = InvalidTerm
    return
}
if rf.log[args.PrevLogIndex].Term != args.PrevLogTerm {
    reply.ConfilictTerm = rf.log[args.PrevLogIndex].Term
    reply.ConfilictIndex = rf.firstIndexFor(reply.ConfilictTerm)
    return
}
```

<font style="color:rgb(51, 51, 51);">Leader 端使用上面两个新增字段的算法如下：</font>

1. <font style="color:rgb(51, 51, 51);">如果 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 为空，说明 Follower 日志太短，直接将 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">nextIndex</font>`<font style="color:rgb(51, 51, 51);"> 赋值为 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictIndex</font>`<font style="color:rgb(51, 51, 51);"> 迅速回退到 Follower 日志末尾</font>**<font style="color:rgb(51, 51, 51);">。</font>**
2. <font style="color:rgb(51, 51, 51);">否则，以 Leader 日志为准，跳过 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 的所有日志；如果发现 Leader 日志中不存在 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictTerm</font>`<font style="color:rgb(51, 51, 51);"> 的任何日志，则以 Follower 为准跳过 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConflictTerm</font>`<font style="color:rgb(51, 51, 51);">，即使用 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">ConfilictIndex</font>`<font style="color:rgb(51, 51, 51);">。</font>

```go
// --- rf.startReplication.replicateToPeer in raft_replication.go
if !reply.Success {
    prevNext := rf.nextIndex[peer]
    if reply.ConfilictTerm == InvalidTerm {
        rf.nextIndex[peer] = reply.ConfilictIndex
    } else {
        firstTermIndex := rf.firstIndexFor(reply.ConfilictTerm)
        if firstTermIndex != InvalidIndex {
            rf.nextIndex[peer] = firstTermIndex + 1
        } else {
            rf.nextIndex[peer] = reply.ConfilictIndex
        }
    }
    // avoid the late reply move the nextIndex forward again
    rf.nextIndex[peer] = MinInt(prevNext, rf.nextIndex[peer])
    return
}
```

<font style="color:rgb(51, 51, 51);">至于我们如何表示空 term 和空 index 呢？</font>

1. **<font style="color:rgb(51, 51, 51);">空 term</font>**<font style="color:rgb(51, 51, 51);">：可以在 make Raft 时让 term 从 1 开始，则 0 就空了出来，可以用来表示空 term，在代码里叫 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">InvalidTerm</font>`
2. **<font style="color:rgb(51, 51, 51);">空 index</font>**<font style="color:rgb(51, 51, 51);">：还记得我们在 rf.log 起始加了一个空 entry 吗？由于这个小技巧，我们的有效日志也是永远从 1 开始，0 就可以用来标识空 index，代码中叫 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">InvalidIndex</font>`

```go
// --- in raft.go
const (
    InvalidIndex int = 0
    InvalidTerm  int = 0
)
```

<font style="color:rgb(51, 51, 51);">为了让代码看着更易懂，我们封装了一个在日志数组中找指定 term 第一条日志的函数</font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">rf.firstLogFor</font>`<font style="color:rgb(51, 51, 51);">：</font>

```go
// --- in raft.go
func (rf *Raft) firstLogFor(term int) int {
    for i, entry := range rf.log {
        if entry.Term == term {
            return i
        } else if entry.Term > term {
            break
        }
    }
    return InvalidIndex
}
```

<font style="color:rgb(51, 51, 51);">由于从前往后，日志的 term 是一段段的单调递增的，则从前往后找，找到第一个满足 term 的日志，就可以返回。如果相关 term 不存在，则返回 </font>`<font style="color:rgb(51, 51, 51);background-color:#FFFFFF;">InvalidIndex</font>`<font style="color:rgb(51, 51, 51);">。</font>


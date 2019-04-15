---
title: 100天阅读计划 -- Day 5
date: 2019-04-15 14:41:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|14:41 | |

> In Search of an Understandable Consensus Algorithm (Raft)


# Introduction

novel features:
- Strong Leader
- Leader election
- Membership changes

# Replicated state machines

Consensus algorithms for practical systems typically have the following properties:
- They ensure safety (never returning an incorrect result) under all non-Byzantine conditions, including network delays, partitions, and packet loss, duplication, and reordering.
- They are fully functional (available) as long as any majority of the servers are operational and can communicate with each other and with clients. Thus, a typical cluster of five servers can tolerate the failure of any two servers. Servers are assumed to fail by stopping; they may later recover from state on stable storage and rejoin the cluster.
- They do not depend on timing to ensure the consis consistency of the logs: faulty clocks and extreme message delays can, at worst, cause availability problems.
- In the common case, a command can complete as soon as a majority of the cluster has responded to a single round of remote procedure calls; a minority of slow servers need not impact overall system performance.

# What’s wrong with Paxos?

- The first drawback is that Paxos is exceptionally difficult to understand.
- The second problem with Paxos is that it does not provide a good foundation for building practical implementations.

# Designing for understandability

- The first technique is the well-known approach of problem decomposition
- Our second approach was to simplify the state space by reducing the number of states to consider, making the system more coherent and eliminating nondeterminism where possible.

# The Raft consensus algorithm

Raft decomposes the consensus problem into three relatively independent subproblems
- Leader election
- Log replication
- Safety

{% asset_img Figure3.png Figure3 %}

## Raft basics

{% asset_img Figure4.png Figure4 %}

- Followers are passive: they issue no requests on their own but simply respond to requests from leaders and candidates. 
- The leader handles all client requests (if a client contacts a follower, the follower redirects it to the leader). 
- The third state, candidate, is used to elect a new leader as described in Section 5.2.

{% asset_img Figure5.png Figure5 %}

Raft divides time into terms of arbitrary length, as shown in Figure 5.

## Leader election



--------

{% raw %}
<iframe src="//player.bilibili.com/player.html?aid=8506617&cid=14004931&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
{% endraw %}


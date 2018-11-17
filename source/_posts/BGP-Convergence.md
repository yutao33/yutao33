---
title: 理解BGP收敛
date: 2018-11-17 14:45:40
mathjax: true
tags: BGP
---

参考 https://blog.ine.com/2010/11/22/understanding-bgp-convergence

## 介绍

BGP是目前域间路由使用的协议，虽然已经提出很多替代协议，没有一个能成功取代。为了增加BGP的广泛适用性，MP-BGP扩展允许BGP传输任何控制平面的信息，比如可以对MPLS/BGP VPN提供自动发现功能或者控制平面互通。但是BGP有个缺点就是收敛速度慢。这篇文章讨论一些技术可以加速BGP的收敛过程。

## BGP收敛过程

BGP是路径矢量协议，换句话说，是具有复杂度量的距离矢量协议。如果不加任何策略，BGP就把AS_PATH属性的长度作为路由的度量值。BGP路由策略可以覆盖这个简单的度量，并可能在非平凡的BGP拓扑中创建分歧条件（参见[7]，[8]，[9]）。虽然这可能在大规模网络中有很严重的问题，但我们不会讨论这些病态案例，而是谈论一般的收敛过程。与任何距离矢量协议一样，BGP路由进程接受多个传入路由更新，并仅向其对等体通告最佳路由。 BGP不使用周期性更新，因此路由失效不是基于任何类型的软状态信息到期（例如，RIP协议中的前缀相关的定时器）。相反，BGP在触发的UPDATE消息中使用显式撤销部分来向邻居发信号通知特定路径的丢失。除了显式提取之外，BGP还支持隐式信令，其中新来的同一对等体的相同前缀的信息替换先前学习的信息。


我们来看看下面的BGP UPDATE消息。 如您所见，UPDATE消息可能包含撤销前缀和新路由信息。 虽然撤销前缀仅被简单地列为NLRI的集合，但新信息被分组在由已宣布的前缀组共享的BGP属性集合周围。 换句话说，每个BGP UPDATE消息包含与一组路径属性有关的新信息，至少有共享相同的AS_PATH属性的前缀。 因此，每个新的属性集合都需要发送单独的UPDATE消息。 这一事实很重要，因为在复制路由信息时，BGP进程会尝试为每个更新消息打包尽可能多的前缀。

{% asset_img BGP-Convergence-FIG0.png BGP-Convergence-FIG0 %}


请看下面的示例拓扑。 让我们假设R1到R7的会话刚刚出现并且前缀20.0.0.0/8通过AS 300传播。在本讨论过程中，我们跳过与BGP策略应用相关的复杂性，忽略了用于处理在运行最佳路径选择过程之前从对等方获知的前缀的BGP Adj-RIB-In空间。( In the course of this discussion we skip the complexities associated with BGP policy application and thus ignore the existence of BGP Adj-RIB-In space used for processing the prefixes learned from a peer prior to running the best-path selection process.)

{% asset_img BGP-Convergence-FIG11.png BGP-Convergence-FIG11 %}

- BGP会话建立并交换BGP OPEN消息后，R1进入“BGP只读模式”。 这意味着，在从R7接收到所有前缀或达到BGP只读模式超时之前，R1不会启动BGP最佳路径选择过程。 超时时间可以使用命令`bgp update-delay`设置。 延迟BGP路径选择过程是为了确保对等方提供了所有路由信息。 这可以最小化最佳路径选择过程运行的次数，简化更新生成并确保更好的前缀消息打包，从而提高传输效率。
- BGP进程以两种方式确定UPDATE消息流的结束：接收BGP KEEPALIVE消息或接收BGP End-of-RIB消息。 后一条消息通常用于BGP平稳重启（参见[13]），但也可用于明确表示BGP UPDATE交换过程的结束。 即使BGP进程不支持End-of-RIB标记，Cisco的BGP实现也会在完成向对等体发送更新时发送KEEPALIVE消息。 很明显，如果对等方需要交换更大的路由表，或者底层TCP传输和路由器入口队列设置使交换更慢，则最佳路径选择延迟会更长。 为解决此问题，我们稍后将简要介绍TCP传输优化。
- 当R1的BGP进程离开只读模式时，它启动最佳路径选择。 此过程遍历新信息并将其与本地BGP RIB内容进行比较，为每个前缀选择最佳路径。 它需要的时间与学习的新信息量成正比。 幸运的是，计算不是非常耗费CPU，就像使用任何距离矢量协议一样。 一旦最佳路径进程完成，BGP必须将所有路由上传到RIB，然后再将它们通告给对等体。 这是距离矢量协议的要求 - 在进一步传播之前使路由信息在RIB中有效。 如果平台支持分布式转发，则RIB更新将依次触发FIB信息上传到路由器的线路卡(line-cards)。 RIB和FIB更新都很耗时，并且花费的时间与更新前缀的数量成正比。
- 在将信息提交到RIB之后，R1需要将最佳路径复制到应该接收它的每个对等方。复制过程可能占用大量内存和CPU，因为BGP进程必须为每个对等体执行完整的BGP表遍历，并为相应的BGP Adj-RIB-Out构造输出。在更新批量计算过程中，这可能需要额外的瞬态内存。但是，Cisco的BGP实现已经通过动态更新组(dynamic update groups)优化了更新生成过程。动态更新组的本质是BGP进程动态查找共享相同输出策略的所有邻居，然后选择具有最低IP地址的对等体作为组长，并仅为组长生成更新批处理。同一组中的所有其他成员都会收到相同的更新。在我们的例子中，R1必须生成两个更新集：一个用于R5，另一个用于RR1和RR2路由反射器。 BGP更新组在路由反射器上变得非常有效，路由反射器通常有数百个对等体共享相同的策略。您可以使用命令`show ip bgp replication`查看IPv4会话的更新组。
- R1开始向R1和RR1，RR2发送更新。这将花费一些时间，具体取决于BGP TCP传输设置和BGP表大小。但是，在R1开始向任何对等/更新组发送任何更新之前，它会检查此对等方是否正在运行Advertisement Interval计时器。 BGP speaker每次完成向对等体发送完整批次更新时，每个对等体启动此计时器。如果准备发送后续批处理而计时器仍在运行，则更新将延迟到计时器到期为止。这是一种阻止机制，可以防止不稳定的对等体通过更新来淹没网络。定义此计时器的命令是` neighbor X.X.X.X advertisment-interval XX`。对于eBGP，默认值为30秒，对于iBGP / eiBGP会话（AS内），默认值为5秒。该计时器真正作用于“Down-Up”或“Up-Down”收敛过程，因为任何快速的拍打变化都会延迟advertisement-interval的时间。这对于AS间路由传播尤为重要，其中默认advertisement-interval间隔为30秒








## 参考资料

[0]RFC4271: Border Gateway Protocol
[1]Advanced BGP Convergence Techniques
[2]Graph Overlays on Path Vector: A Possible Next Step in BGP
[3]BGP Add Paths Capability
[4]BGP Convergence in much less than a second
[5]BGP PIC Configuration Guide
[6]OSPF Fast Convergence
[7]An Analysis of BGP Convergence Properties
[8]RFC4451: BGP MULTI_EXIT_DISC (MED) Considerations
[9]RFC3345: Border Gateway Protocol (BGP) Persistent Route Oscillation Condition
[10]BGP Design and Implementation by Randy Zhang
[11]RFC 4274: BGP Protocol Analysis
[12]Day in the Life of a BGP Update in Cisco IOS
[13]RFC 4724: Graceful Restart for BGP
[14]Optimizing IP Event Dampening
[15]RFC 5883: Multihop BFD
[16]BGP Add Path Overview
[17]BGP Add Paths Scaling/Performance Tradeoffs
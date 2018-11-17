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

该过程在RR1和RR2上重复，从传入的UPDATE数据包接收，最佳路径选择和更新生成开始。 如果由于某种原因，前缀20.0.0.0/8将在宣布后立即从AS 100消失，可能需要与“Number_of_Hops x Advertisement_Interval”一样长的时间到达R3和R4，因为每一跳可能会延迟快速后续更新。 我们可以看到，BGP收敛的主要限制因素是BGP表大小，传输级设置和宣告延迟。 最佳路径选择时间与表大小以及更新批处理所需的时间成比例。

让我们看一个稍微不同的场景来演示BGP多路径如何潜在地改善收敛。首先，观察图1中呈现的拓扑，我们可能注意到AS 300具有到AS 100的两个连接。因此，可以预期在AS 300中的每个路由器上看到到AS 100的每条路径的两条路径。但是这不是在AS内部使用除BGP全网格之外的任何拓扑的情况下，总是可能的。在我们的示例中，R1和R2将路由信息通告给路由反射器RR1和RR2。根据距离矢量行为，反射器将仅重新通告AS 100前缀的最佳路径，并且由于两个RR一致地选择路径，因此它们将向R3，R4和R2通告相同的路径。 R3和R4都将从每个RR接收前缀10.0.0.0/24，并通过R1使用路径。 R2也将通过R1获得最佳路径，但更喜欢使用其eBGP连接。相反，如果R1，R2，R3和R4连接在全网状网中，那么每个路由器都会看到通过R1和R2的出口，并且如果配置了则能够使用BGP多路径。让我们回顾一下当R1失去与AS 100的连接时图1中的拓扑结构会发生什么。

- 根据故障检测机制，无论是BGP keepalive还是BFD，R1都需要一段时间才能实现连接不再有效。 我们将在本出版物后面讨论快速故障检测的选项。
- 在意识到R5消失后，R1通过R7删除所有路径。 由于RR1和RR2从未通过R2向R1返回路径，因此R1没有到AS100的备用路径。为实现这一点，R1为RR1，RR2和R7准备了一批UPDATE消息，其中包含AS100前缀的提取消息。 一旦RR1和RR2完成接收和处理提款，他们就通过R2选择新的最佳路径并向R1，R2，R3，R4通告提款/更新。
- R3和R4现在具有通过R2的新路径，并且R2通过它从RR知道的R1失去“备份”路径。 在这种情况下，重新收敛过程的主要工作是路线反射器。 收敛时间是RR中对等会话失败检测，更新通告和BGP最佳路径重新计算的总和。

如果BGP发言者能够同时使用多条路径，则可以减轻网络故障的严重性。 实际上，如果正在使用负载平衡，那么退出点的故障将只影响流经该出口点的流量（在我们的情况下为50％），并且只有那些流量必须等待重新收敛时间。 更好的是，理论上可以在BGP中有多个等成本（等效且因此无环路径）路径的情况下进行“快速”重新路由。 一旦发出故障信号，就可以在转发引擎中执行这种切换。 但是，这种类型的重新路由机制存在两个主要问题：

1. 正如我们所看到的，路由反射器（或联合）的使用通过隐藏备用路径对冗余有显着影响。 使用全网格不是一种选择，因此需要一种机制来允许在RR /联盟环境中传播多个备用路径。 有趣的是，这种机制已经在BGP / MPLS VPN场景中可用，其中CE站点的多个附着点可以利用不同的RD值来区分从不同连接点通告的相同路由。 但是，需要通用解决方案，允许使用IPv4或任何其他地址系列通告多个备用路径。
2. 通过BGP机制进行故障检测和传播的速度很慢，并且取决于受影响前缀的数量。 因此，损坏越严重，它在BGP中传播的速度就越慢。 需要使用其他一些非BGP机制来报告网络故障并触发BGP重新收敛。

在接下来的部分中，我们将回顾为加速BGP收敛而开发的各种技术，与“基于纯BGP”的故障检测和修复相比，可以实现更好的响应时间。

--待续--



## 参考资料

[[0]RFC4271: Border Gateway Protocol](http://tools.ietf.org/html/rfc4271)
[[1]Advanced BGP Convergence Techniques](http://www.apricot.net/apricot2006/slides/conf/wednesday/Pradosh_Mohapatra-apricot-2006.ppt)
[[2]Graph Overlays on Path Vector: A Possible Next Step in BGP](http://www.cisco.com/web/about/ac123/ac147/archived_issues/ipj_8-2/graph_overlays.html)
[[3]BGP Add Paths Capability](http://tools.ietf.org/html/draft-walton-bgp-add-paths-06)
[[4]BGP Convergence in much less than a second](http://www.nanog.org/meetings/nanog40/abstracts.php?pt=MjM1Jm5hbm9nNDA=&nm=nanog40)
[[5]BGP PIC Configuration Guide](http://www.cisco.com/en/US/docs/ios/iproute_bgp/configuration/guide/irg_bgp_mp_pic_ps6922_TSD_Products_Configuration_Guide_Chapter.html)
[[6]OSPF Fast Convergence](https://blog.ine.com/2010/06/02/ospf-fast-convergenc/)
[[7]An Analysis of BGP Convergence Properties](http://www.csci.csusb.edu/ykarant/courses/w2005/csci531/papers/p277-griffin.pdf)
[[8]RFC4451: BGP MULTI_EXIT_DISC (MED) Considerations](http://www.ietf.org/rfc/rfc4451.txt)
[[9]RFC3345: Border Gateway Protocol (BGP) Persistent Route Oscillation Condition](http://www.ietf.org/rfc/rfc3345.txt)
[[10]BGP Design and Implementation by Randy Zhang]()
[[11]RFC 4274: BGP Protocol Analysis](http://tools.ietf.org/html/rfc4274)
[[12]Day in the Life of a BGP Update in Cisco IOS](http://www.ripe.net/ripe/meetings/ripe-45/presentations/ripe45-routing-bgp-update.pdf)
[[13]RFC 4724: Graceful Restart for BGP](http://tools.ietf.org/html/rfc4724)
[[14]Optimizing IP Event Dampening](https://blog.ine.com/tag/ip-event-dampening/)
[[15]RFC 5883: Multihop BFD](http://tools.ietf.org/html/rfc5883s)
[[16]BGP Add Path Overview](https://blog.ine.com/2010/11/22/www.nanog.org/meetings/nanog48/presentations/Tuesday/Ward_AddPath_N48.pdf)
[[17]BGP Add Paths Scaling/Performance Tradeoffs](http://inl.info.ucl.ac.be/publications/bgp-add-paths-scalingperformance-tradeoffs)


## APPENDIX: Practical Scenario Baseline Configuration

The below are the initial configurations for the Dynamips topology used to validate BGP PIC behavior.

```
====R1:====
hostname R1
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 ip address 20.0.17.1 255.255.255.0
 no shut
!
interface Serial 1/2
 no shut
 ip address 10.0.12.1 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.13.1 255.255.255.0
!
interface Serial 1/3
 ip address 10.0.14.1 255.255.255.0
!
interface Loopback0
 ip address 10.0.1.1 255.255.255.255
!
router ospf 100
 router-id 10.0.1.1
 network 0.0.0.0 0.0.0.0 area 0
 passive-interface Serial 1/0
!
router bgp 100
 neighbor 10.0.3.3 remote-as 100
 neighbor 10.0.3.3 update-source Loopback0
 neighbor 10.0.4.4 remote-as 100
 neighbor 10.0.4.4 update-source Loopback0
 neighbor 20.0.17.7 remote-as 200
====R2:====
hostname R2
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 ip address 20.0.28.2 255.255.255.0
 no shut
!
interface Serial 1/2
 no shut
 ip address 10.0.12.2 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.24.2 255.255.255.0
!
interface Serial 1/3
 no shut
 ip address 10.0.23.2 255.255.255.0
!
interface Loopback0
 ip address 10.0.2.2 255.255.255.255
!
router ospf 100
 router-id 10.0.2.2
 network 0.0.0.0 0.0.0.0 area 0
 passive-interface Serial 1/0
!
router bgp 100
 neighbor 10.0.3.3 remote-as 100
 neighbor 10.0.3.3 update-source Loopback0
 neighbor 10.0.4.4 remote-as 100
 neighbor 10.0.4.4 update-source Loopback0
 neighbor 20.0.28.8 remote-as 200
====R3:====
hostname R3
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 10.0.13.3 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.35.3 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 10.0.34.3 255.255.255.0
!
interface Serial 1/3
 no shut
ip address 10.0.23.3 255.255.255.0
!
interface Serial 1/4
 no shut
 ip address 10.0.36.3 255.255.255.0
!
router ospf 100
 router-id 10.0.3.3
 network 0.0.0.0 0.0.0.0 area 0
!
interface Loopback0
 ip address 10.0.3.3 255.255.255.255
!
router bgp 100
 neighbor IBGP peer-group
 neighbor IBGP remote-as 100
 neighbor IBGP update-source Loopback0
 neighbor IBGP route-reflector-client
 neighbor 10.0.1.1 peer-group IBGP
 neighbor 10.0.2.2 peer-group IBGP
 neighbor 10.0.5.5 peer-group IBGP
 neighbor 10.0.6.6 peer-group IBGP
 neighbor 10.0.4.4 remote-as 100
 neighbor 10.0.4.4 update-source Loopback0
====R4:====
hostname R4
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 10.0.24.4 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.46.4 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 10.0.34.4 255.255.255.0
!
interface Serial 1/3
 no shut
ip address 10.0.14.4 255.255.255.0
!
interface Serial 1/4
 no shut
 ip address 10.0.45.4 255.255.255.0
!
router ospf 100
 router-id 10.0.4.4
 network 0.0.0.0 0.0.0.0 area 0
!
interface Loopback0
 ip address 10.0.4.4 255.255.255.255
!
router bgp 100
 neighbor IBGP peer-group
 neighbor IBGP remote-as 100
 neighbor IBGP update-source Loopback0
 neighbor IBGP route-reflector-client
 neighbor 10.0.1.1 peer-group IBGP
 neighbor 10.0.2.2 peer-group IBGP
 neighbor 10.0.5.5 peer-group IBGP
 neighbor 10.0.6.6 peer-group IBGP
 neighbor 10.0.3.3 remote-as 100
 neighbor 10.0.3.3 update-source Loopback0
====R5:====
hostname R5
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 10.0.35.5 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.56.5 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 10.0.45.5 255.255.255.0
!
router ospf 100
 router-id 10.0.5.5
 network 0.0.0.0 0.0.0.0 area 0
!
interface Loopback0
 ip address 10.0.5.5 255.255.255.0
!
router bgp 100
 neighbor 10.0.3.3 remote-as 100
 neighbor 10.0.3.3 update-source Loopback0
 neighbor 10.0.4.4 remote-as 100
 neighbor 10.0.4.4 update-source Loopback0
====R6:====
hostname R6
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 10.0.46.6 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 10.0.56.6 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 10.0.36.6 255.255.255.0
!
router ospf 100
 router-id 10.0.6.6
 network 0.0.0.0 0.0.0.0 area 0
!
interface Loopback0
 ip address 10.0.6.6 255.255.255.0
!
router bgp 100
 neighbor 10.0.3.3 remote-as 100
 neighbor 10.0.3.3 update-source Loopback0
 neighbor 10.0.4.4 remote-as 100
 neighbor 10.0.4.4 update-source Loopback0 
====R7:====
hostname R7
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 20.0.17.7 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 20.0.78.7 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 20.0.79.7 255.255.255.0
!
interface Loopback0
 ip address 20.0.7.7 255.255.255.0
!
router ospf 1
 router-id 20.0.7.7
 network 0.0.0.0 0.0.0.0 area 0
 passive-interface Serial 1/0
!
router bgp 200
 neighbor 20.0.17.1 remote-as 100
 neighbor 20.0.9.9 remote-as 200
 neighbor 20.0.9.9 update-source Loopback0
 neighbor 20.0.8.8 remote-as 200
 neighbor 20.0.8.8 update-source Loopback0
====R8:====
hostname R8
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 20.0.28.8 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 20.0.78.8 255.255.255.0
!
interface Serial 1/2
 no shut
 ip address 20.0.89.8 255.255.255.0
!
interface Loopback0
 ip address 20.0.8.8 255.255.255.0
!
router ospf 1
 router-id 20.0.8.8
 network 0.0.0.0 0.0.0.0 area 0
 passive-interface Serial 1/0
!
router bgp 200
 neighbor 20.0.28.2 remote-as 100
 neighbor 20.0.9.9 remote-as 200
 neighbor 20.0.9.9 update-source Loopback0
 neighbor 20.0.7.7 remote-as 200
 neighbor 20.0.7.7 update-source Loopback0
====R9:====
hostname R9
!
ip tcp synwait-time 5
no ip domain-lookup
no service timestamps
!
line con 0
 logging synch
 exec-timeout 0 0
 privilege level 15
!
ip routing
!
interface Serial 1/0
 no shut
 ip address 20.0.79.9 255.255.255.0
!
interface Serial 1/1
 no shut
 ip address 20.0.89.9 255.255.255.0
!
interface Loopback0
 ip address 20.0.9.9 255.255.255.0
!
interface Loopback100
 ip address 20.0.99.99 255.255.255.0
!
router ospf 1
 router-id 20.0.9.9
 network 0.0.0.0 0.0.0.0 area 0
!
router bgp 200
 neighbor 20.0.8.8 remote-as 200
 neighbor 20.0.8.8 update-source Loopback0
 neighbor 20.0.7.7 remote-as 200
 neighbor 20.0.7.7 update-source Loopback0
 network 20.0.99.0 mask 255.255.255.0
```
---
title: Understanding BGP MED Attribute
date: 2018-11-19 15:36:40
mathjax: true
tags: BGP
---

参考 <https://www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/112965-bgpmed-attr-00.html>

## 介绍

MED (Multi Exit Discriminator, 多出口鉴别器)提供了一种动态的方法可以影响另一个AS到达某一个特定的路由。BGP遵循一个系统的过程去选择最佳路由，在使用MED属性之前，也有其他比如权重(weight)、局部优先级(local preference)、起源路由(originate route)、AS路径(AS path)等属性被用来决策，一旦这些条件有满足的，MED属性就不会被考虑。

注意： 当所有其他因素相等时，具有最低MED的出口点是首选

## 示例

### 场景1

当一个BGP speaker从一个对等体学习到路由的时候，这个路由的MED会传递到其他内部BGP(iBGP)对等体，但不会传递到外部BGP(eBGP)对等体。

{% asset_img 112965-bgpmed-attr-01.gif 112965-bgpmed-attr-01 %}

Router 1

```sh
(Config)#interface Loopback10
(Config-if)#ip address 10.10.10.10 255.255.255.255
(Config-if)#interface FastEthernet0/0
(Config-if)#ip address 192.1.12.1 255.255.255.0
(Config)#router bgp 100
(Config-router)#no synchronization
(Config-router)#bgp router-id 10.10.10.10
(Config-router)#bgp log-neighbor-changes
(Config-router)#network 10.10.10.10 mask 255.255.255.255 route-map ATTACH_MED
(Config-router)#neighbor 192.1.12.2 remote-as 100
(Config-router)#no auto-summary
(Config)#access-list 10 permit 10.10.10.10
(Config)#route-map ATTACH_MED permit 10
(Config)#match ip address 10
(Config)#set metric 100
```

Router 2

```sh
(Config)#interface FastEthernet0/0
(Config-if)#ip address 192.1.12.2 255.255.255.0
(Config-if)#interface Serial1/0
(Config-if)#ip address 192.1.23.2 255.255.255.0
(Config-if)#encapsulation frame-relay IETF
(Config-if)#no fair-queue
(Config-if)#frame-relay map ip 192.1.23.3 203 broadcast
(Config-if)#no frame-relay inverse-arp
(Config-if)#frame-relay lmi-type ansi
(Config)#router bgp 100
(Config-router)#no synchronization
(Config-router)#bgp router-id 22.22.22.22
(Config-router)#bgp log-neighbor-changes
(Config-router)#neighbor 192.1.12.1 remote-as 100
(Config-router)#neighbor 192.1.23.3 remote-as 101
(Config-router)#neighbor 192.1.23.3 ebgp-multihop 3
(Config-router)#no auto-summary
```

Router 3

```sh
(Config)#interface Serial1/0
(Config-if)#ip address 192.1.23.3 255.255.255.0
(Config-if)#encapsulation frame-relay IETF
(Config-if)#no fair-queue
(Config-if)#frame-relay map ip 192.1.23.2 302 broadcast
(Config-if)#no frame-relay inverse-arp
(Config-if)#frame-relay lmi-type ansi
(Config)#router bgp 101
(Config-router)#no synchronization
(Config-router)#bgp log-neighbor-changes
(Config-router)#neighbor 192.1.23.2 remote-as 100
(Config-router)#neighbor 192.1.23.2 ebgp-multihop 3
(Config-router)#no auto-summary
```

在这个设置之下，R1和R2在运行iBGP，因此，当一个更新带着一个特定的Metric进入AS这个AS时，这个Metric会被用来在这个AS内做决策。用`show ip bgp`命令检查R2的路由可以显示出对于10.10.10.10的metric值，这是从iBGP邻居192.168.12.1学习到的，并且有一个MED值100

R2的输入如下图

{% asset_img 112965-bgpmed-attr-02.gif 112965-bgpmed-attr-02 %}

R2和R3之间运行eBGP，当同样的更新到达另一个AS时，这个Metric归零，`show ip bgp`检查R3可以看出它的Metric是0

R3的输出如下图

{% asset_img 112965-bgpmed-attr-03.gif 112965-bgpmed-attr-03 %}

从这一场景中可以明显看出MED属性可以影响邻居AS进来的流量，但不会影响其他AS。当一个BGP speaker从一个对等体学习到路由的时候，他可以将这条路由的MED传递到任何iBGP对等体，但不会是eBGP对等体。结果就是，MED只能影响邻居AS。

### 场景2

当注入BGP的路由(通过使用`network`或`redistribute`命令)来自于其他IGP(RIP、EIGRP、OSFPF)，这里MED来自于IGP metric并且带有这个MED的路由被宣告到eBGP邻居。

{% asset_img 112965-bgpmed-attr-04.gif 112965-bgpmed-attr-04 %}

Router R1

```sh
(Config)#interface Loopback10
(Config-if)#ip address 10.10.10.10 255.255.255.255
(Config-if)#interface FastEthernet0/0
(Config-if)#ip address 192.1.12.1 255.255.255.0
(Config)#router rip
(Config-router)#network 10.0.0.0
(Config-router)#network 192.1.12.0
(Config-router)#no auto-summary
```

路由器R2和R3上配置BGP, R2上配置redistribution以把RIP网络注入BGP

Router R2

```sh
(Config)#interface FastEthernet0/0
(Config-if)#ip address 192.1.12.2 255.255.255.0
(Config-if)#interface Serial1/0
(Config-if)#ip address 192.1.23.2 255.255.255.0
(Config-if)#encapsulation frame-relay IETF
(Config-if)#no fair-queue
(Config-if)#frame-relay map ip 192.1.23.3 203 broadcast
(Config-if)#no frame-relay inverse-arp
(Config-if)#frame-relay lmi-type ansi
(Config)#router rip
(Config-router)# network 192.1.12.0
(Config-router)#no auto-summary
(Config-router)#router bgp 100
(Config-router)#no synchronization
(Config-router)#bgp router-id 22.22.22.22
(Config-router)#bgp log-neighbor-changes
(Config-router)#neighbor 192.1.23.3 remote-as 101
(Config-router)#neighbor 192.1.23.3 ebgp-multihop 3
(Config-router)#redistribute rip metric 1
(Config-router)#no auto-summary
```

Router R3

```sh
(Config)#interface Serial1/0
(Config-if)#ip address 192.1.23.3 255.255.255.0
(Config-if)#encapsulation frame-relay IETF
(Config-if)#no fair-queue
(Config-if)#frame-relay map ip 192.1.23.2 302 broadcast
(Config-if)#no frame-relay inverse-arp
(Config-if)#frame-relay lmi-type ansi
(Config)#router bgp 101
(Config-router)# no synchronization
(Config-router)#bgp router-id 33.33.33.33
(Config-router)#bgp log-neighbor-changes
(Config-router)#neighbor 192.1.23.2 remote-as 100
(Config-router)#neighbor 192.1.23.2 ebgp-multihop 3
(Config-router)#no auto-summary
```

在R2上用`show ip bgp`命令可以看到10.0.0.0网络的metric是1，这是来自于RIP协议

R2的输出如下

{% asset_img 112965-bgpmed-attr-05.gif 112965-bgpmed-attr-05 %}

However, in R3 which runs on eBGP, the network is advertised by considering the MED value derived from the IGP. In this case it is RIP. The prefix 10.0.0.0 is advertised with the IGP MED value, which is RIP's metric 1.

R3的输出如下

{% asset_img 112965-bgpmed-attr-06.gif 112965-bgpmed-attr-06 %}

From this scenario the behavior of the MED, in the case of networks being injected to the BGP router via the `network` or `redistribute` command, is clearly seen where the actual MED value is being replaced with that of the IGP metric. Now, given that this attribute is a hint to external neighbors about the path preference into an AS. As stated earlier, it is not always considered if there are other more important attributes to determine the best route. In order to have the same effect with a more deterministic attribute, use the set as-path prepend command under the route map. If you prepend the AS path for certain routes, it will continue to be seen by other AS. For more information on the usage of As-path prepend, refer to [Use of Set-aspath prepend Command](https://www.cisco.com/en/US/tech/tk365/technologies_tech_note09186a00800c95bb.shtml#neighborsroutemaps).
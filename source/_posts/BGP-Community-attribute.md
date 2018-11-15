---
title: BGP Community attributes
date: 2018-11-15 16:31:40
mathjax: true
tags: BGP
---

本文参考 http://network-101.blogspot.com/2011/01/bgp-community-attribute.html

BGP社区是BGP路由协议的一个可选属性，可以认为它是一个IGP的tag，表示一个IP地址集合。社区标签常用来维护路由、设置一个IP集合的BGP属性。社区属性设置后，可以被路由映射函数(route-map function)改变。默认情况下，社区标签是一个十进制数，但是我们可以通过`ip bgp-community new-format`使用新的格式(AA:NN)，AA是自治系统号，NN是ASN设置的值。

有三个特殊的社区属性值。命令名称和功能如下表所示


| Name     | Value    | Meaning |
|----------|----------|---------|
|NO_EXPORT| FFFF:FF01| Do not advertise outside this AS. It can be advertised to other confederation autonomous systems.|
|NO_ADVERT| FFFF:FF02| Do not advertise to any other peer.|
|LOCAL_AS | FFFF:FF03| Do not advertise outside the local confederation sub-AS.|

要设置BGP社区属性，可以在`route-map`命令中使用`set community <number>/<special community string>`，或者可以使用`ip community-list <community string> permit/deny <ACL number>`。

下面是一个配置BGP社区属性的例子，我们不讨论一个十进制的社区属性，因为它和IGP tag 起相同的作用。在这个例子里, R1,R2和R3在AS 100里，R4在 AS 200里。 OSPF在R1,R2和R3之间运行。下面是这个例子的配置要求。

- 在R1和R4之间建立eBPG连接
- 在R1、R2和R3之间使用路由反射器建立iBGP连接
- R3使用图中的社区属性在BGP里宣告loopback网络
- 验证R2、R1和R4上的路由和社区属性

{% asset_img BGP-Community-attribute.jpg Example %}


在这个例子里，我们期望

- R2不会有`150.150.150.0/24`子网的信息（R1不会向它的peers宣告路由）
- `100.100.100.0/24`不会向AS 100之外宣告路由
- `200.200.200.0/24`正常宣告


### 设置R3

配置R3用图中的社区属性宣告它的loopback网络

### 对每个loopback接口设置access-list

```sh
ip access-list standard only-100
 permit 100.100.0.0 0.0.255.255

ip access-list standard only-150
 permit 150.150.0.0 0.0.255.255

ip access-list standard only-200
 permit 200.200.0.0 0.0.255.255
```

### 对路由映射(route-map)设置access-list, 根据图中配置设置社区属性

```sh
route-map communityset permit 10
 match ip address only-100
 set community no-export
!
route-map communityset permit 20
 match ip address only-150
 set community no-advertise
!
route-map communityset permit 30
 set community none
```

### 对BGP会话设置"communityset"策略

```sh
router bgp 100
 no synchronization
 bgp log-neighbor-changes
 network 100.100.100.0 mask 255.255.255.0
 network 150.150.150.0 mask 255.255.255.0
 network 200.200.200.0
 neighbor 1.1.1.1 remote-as 100
 neighbor 1.1.1.1 update-source Loopback0
 neighbor 1.1.1.1 send-community both
 neighbor 1.1.1.1 route-map communityset out
 no auto-summary
```

注意:

1. `neighbor send community`需要应用到邻居以发送社区属性
2. 对邻居应用路由映射(route-map)

### R1社区属性验证

```sh
R1#sh ip bgp 100.100.100.0
BGP routing table entry for 100.100.100.0/24, version 24
Paths: (1 available, best #1, table Default-IP-Routing-Table, not advertised to EBGP peer)
  Advertised to update-groups:
        2
  Local, (Received from a RR-client)
    3.3.3.3 (metric 21) from 3.3.3.3 (3.3.3.3)
      Origin IGP, metric 0, localpref 100, valid, internal, best
      Community: no-export
R1#
R1#sh ip bgp 150.150.150.0
BGP routing table entry for 150.150.150.0/24, version 25
Paths: (1 available, best #1, table Default-IP-Routing-Table, not advertised to any peer)
  Not advertised to any peer
  Local, (Received from a RR-client)
    3.3.3.3 (metric 21) from 3.3.3.3 (3.3.3.3)
      Origin IGP, metric 0, localpref 100, valid, internal, best
      Community: no-advertise
R1#
```

### 路由表验证

#### 1) R2的路由表里没有`150.150.150.0/24`

```sh
R2 Routing table 

B    200.200.200.0/24 [200/0] via 3.3.3.3, 16:28:17
     1.0.0.0/32 is subnetted, 1 subnets
O       1.1.1.1 [110/11] via 192.168.1.1, 16:54:32, FastEthernet0/0
     2.0.0.0/32 is subnetted, 1 subnets
C       2.2.2.2 is directly connected, Loopback0
     100.0.0.0/24 is subnetted, 1 subnets
B       100.100.100.0 [200/0] via 3.3.3.3, 16:52:28
     3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/11] via 192.168.2.2, 16:53:54, FastEthernet0/1
     10.0.0.0/24 is subnetted, 1 subnets
O       10.1.1.0 [110/20] via 192.168.1.1, 16:54:32, FastEthernet0/0
     123.0.0.0/24 is subnetted, 1 subnets
B       123.1.1.0 [200/0] via 1.1.1.1, 16:49:44
C    192.168.1.0/24 is directly connected, FastEthernet0/0
C    192.168.2.0/24 is directly connected, FastEthernet0/1
R2#
```

#### 2) R4路由表里没有`100.100.100.0/24`

```sh
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

B    200.200.200.0/24 [20/0] via 10.1.1.1, 16:30:36
     10.0.0.0/24 is subnetted, 1 subnets
C       10.1.1.0 is directly connected, FastEthernet0/0
     123.0.0.0/24 is subnetted, 1 subnets
C       123.1.1.0 is directly connected, Loopback0
R4#
```


### R1的路由表

Gateway of last resort is not set.

```sh
B    200.200.200.0/24 [200/0] via 3.3.3.3, 16:31:09
     1.0.0.0/24 is subnetted, 1 subnets
C       1.1.1.0 is directly connected, Loopback0
     2.0.0.0/32 is subnetted, 1 subnets
O       2.2.2.2 [110/11] via 192.168.1.2, 16:57:33, FastEthernet0/1
     100.0.0.0/24 is subnetted, 1 subnets
B       100.100.100.0 [200/0] via 3.3.3.3, 16:39:33
     3.0.0.0/32 is subnetted, 1 subnets
O       3.3.3.3 [110/21] via 192.168.1.2, 16:55:43, FastEthernet0/1
     10.0.0.0/24 is subnetted, 1 subnets
C       10.1.1.0 is directly connected, FastEthernet0/0
     123.0.0.0/24 is subnetted, 1 subnets
B       123.1.1.0 [20/0] via 10.1.1.2, 16:52:35
C    192.168.1.0/24 is directly connected, FastEthernet0/1
O    192.168.2.0/24 [110/20] via 192.168.1.2, 16:56:55, FastEthernet0/1
     150.150.0.0/24 is subnetted, 1 subnets
B       150.150.150.0 [200/0] via 3.3.3.3, 16:35:24
R1#
```
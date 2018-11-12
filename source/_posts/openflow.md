---
title: OpenFlow命令示例
date: 2018-11-12 15:30:40
mathjax: true
tags: OpenFlow
---


转自 https://blog.csdn.net/mingge591/article/details/50952305

创建一个OVS交换机

```
ovs-vsctl add-br ovs-switch
```


创建一个端口 p0，设置端口 p0 的 OpenFlow 端口编号为 100

```
ovs-vsctl add-port ovs-switch p0 -- set Interface p0 ofport_request=100
```

设置网络接口设备类型为"internal"

```
ovs-vsctl set Interface p0 type=internal
```


查看设置结果

```
ethtool -i p0
```

```
driver: openvswitch
version: 
firmware-version: 
bus-info: 
supports-statistics: no
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```
 

创建一个name space：ns0，把p0端口接入到ns0里，并且配置ip地址 192.168.1.100/24

```
ip netns add ns0
ip link set p0 netns ns0
ip netns exec ns0 ip addr add 192.168.1.100/24 dev p0
ip netns exec ns0 ifconfig p0 promisc up
```

查看创建结果

```
ovs-vsctl show
6507c214-0c7a-4159-9813-977074f73aa1
    Bridge ovs-switch
        Port "p0"
            Interface "p0"
                type: internal
        Port ovs-switch
            Interface ovs-switch
                type: internal
    ovs_version: "2.0.2"
```


重复步骤，创建p1和p2端口

```
ovs-vsctl add-port ovs-switch p1 -- set Interface p1 ofport_request=101
ovs-vsctl set Interface p1 type=internal
ip netns add ns1
ip link set p1 netns ns1
ip netns exec ns1 ip addr add 192.168.1.101/24 dev p1
ip netns exec ns1 ifconfig p1 promisc up
```

查看创建结果

```
ovs-vsctl show
c402802a-dbe0-4127-9bce-e2405a4ac33d
    Bridge ovs-switch
        Port "p0"
            Interface "p0"
                type: internal
        Port ovs-switch
            Interface ovs-switch
                type: internal
        Port "p1"
            Interface "p1"
                type: internal
    ovs_version: "2.0.2"
```

创建p2

```
ovs-vsctl add-port ovs-switch p2 -- set Interface p2 ofport_request=102
ovs-vsctl set Interface p2 type=internal
ip netns add ns2
ip link set p2 netns ns2
ip netns exec ns2 ip addr add 192.168.1.102/24 dev p2
ip netns exec ns2 ifconfig p2 promisc up
```

```
c402802a-dbe0-4127-9bce-e2405a4ac33d
    Bridge ovs-switch
        Port "p0"
            Interface "p0"
                type: internal
        Port "p2"
            Interface "p2"
                type: internal
        Port ovs-switch
            Interface ovs-switch
                type: internal
        Port "p1"
            Interface "p1"
                type: internal
    ovs_version: "2.0.2"
```

查看创建的交换机信息，获得dpid，端口openflow端口编号

```
ovs-ofctl show ovs-switch
```

```
OFPT_FEATURES_REPLY (xid=0x2): dpid:0000c6d6454c204e
n_tables:254, n_buffers:256
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS ARP_MATCH_IP
actions: OUTPUT SET_VLAN_VID SET_VLAN_PCP STRIP_VLAN SET_DL_SRC SET_DL_DST SET_NW_SRC SET_NW_DST SET_NW_TOS SET_TP_SRC SET_TP_DST ENQUEUE
100(p0): addr:00:00:00:00:00:00
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
101(p1): addr:00:00:00:00:00:00
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
102(p2): addr:00:00:00:00:00:00
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
LOCAL(ovs-switch): addr:76:f1:e5:e8:19:ed
     config:     PORT_DOWN
     state:      LINK_DOWN
     speed: 0 Mbps now, 0 Mbps max
OFPT_GET_CONFIG_REPLY (xid=0x4): frags=normal miss_send_len=0
```

获取openflow端口编号

```
ovs-vsctl get Interface p0 ofport
```

```
100
ovs-vsctl get Interface p1 ofport
```

```
101
ovs-vsctl get Interface p2 ofport
```

```
102


查看 datapath 的信息
ovs-dpctl show
```

```
system@ovs-system:
        lookups: hit:35 missed:21 lost:0
        flows: 0
        port 0: ovs-system (internal)
        port 1: ovs-switch (internal)
        port 2: p0 (internal)
        port 3: p1 (internal)
        port 4: p2 (internal)
```
 

查看mac地址

```
ip netns exec ns0 ping 192.168.1.100
ip netns exec ns0 ping 192.168.1.101
ip netns exec ns0 ping 192.168.1.102
```
这里100ping不同，原因不明

然后运行

```
ovs-appctl fdb/show ovs-switch
```

```
port  VLAN  MAC                Age
  101     0  1a:19:57:65:03:f2   55
  102     0  36:54:7a:1c:c9:43   48
  100     0  36:92:12:98:4d:40   48
```

查看交换机所有table

```
ovs-ofctl dump-tables ovs-switch
```
 

253个table.

查看交换机中的所有流表项

```
ovs−ofctl dump−flows ovs-switch
```
输出如下信息：

```
cookie=0x0, duration=1756.751s, table=0, n_packets=96, n_bytes=7784, idle_age=303, priority=0 actions=NORMAL
```


删除编号为 100 的端口上的所有流表项

```
ovs-ofctl del-flows ovs-switch "in_port=100"
```


查看交换机端口信息

```
ovs-ofctl show ovs-switch
```


修改数据包

屏蔽所有进入 OVS 的以太网广播数据包

```
ovs-ofctl add-flow ovs-switch "table=0, dl_src=01:00:00:00:00:00/01:00:00:00:00:00, actions=drop"
```
屏蔽 STP 协议的广播数据包

```
ovs-ofctl add-flow ovs-switch "table=0, dl_dst=01:80:c2:00:00:00/ff:ff:ff:ff:ff:f0, actions=drop"
```


修改数据包，添加新的 OpenFlow 条目，修改从端口 p0 收到的数据包的源地址为 9.181.137.1

```
ovs-ofctl add-flow ovs-switch "priority=1 idle_timeout=0,\
    in_port=100,actions=mod_nw_src:9.181.137.1,normal"
```

从端口 p0（192.168.1.100）发送测试数据到端口 p1（192.168.1.101），就是没啥响应

```
ip netns exec ns0 ping 192.168.1.101
```


再打开一个ssh终端，登录进去，运行tcpdump，需要等待几分钟，才能看到响应

```
ip netns exec ns1 tcpdump -i p1 icmp
```

```
ip netns exec ns1 tcpdump -i p1 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on p1, link-type EN10MB (Ethernet), capture size 65535 bytes
02:40:20.212575 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 89, length 64
02:40:21.220600 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 90, length 64
02:40:22.228630 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 91, length 64
02:40:23.236670 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 92, length 64
02:40:24.244721 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 93, length 64
02:40:25.252758 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 94, length 64
02:40:26.260777 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 95, length 64
02:40:27.268868 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 96, length 64
02:40:28.276857 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 97, length 64
02:40:29.284839 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 98, length 64
02:40:30.292939 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 99, length 64
02:40:31.300987 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 100, length 64
02:40:32.308972 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 101, length 64
02:40:33.317053 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 102, length 64
02:40:34.325103 IP 9.181.137.1 > 192.168.1.101: ICMP echo request, id 2118, seq 103, length 64
```


重定向数据包

添加新的 OpenFlow 条目，重定向所有的 ICMP 数据包到端口 p2

```
ovs-ofctl add-flow ovs-switch idle_timeout=0,dl_type=0x0800,nw_proto=1,actions=output:102
```
从端口 p0 （192.168.1.100）发送数据到端口 p1（192.168.1.101）

这个时候你从p2里，可以看到

```
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on p2, link-type EN10MB (Ethernet), capture size 65535 bytes
06:25:38.252471 IP 192.168.1.100 > 192.168.1.101: ICMP echo request, id 4668, seq 35, length 64
06:25:39.260438 IP 192.168.1.100 > 192.168.1.101: ICMP echo request, id 4668, seq 36, length 64
06:25:40.268419 IP 192.168.1.100 > 192.168.1.101: ICMP echo request, id 4668, seq 37, length 64
```


修改vlan tag

修改端口 p1 的 VLAN tag 为 101，使端口 p1 成为一个隶属于 VLAN 101 的端口

```
ovs-vsctl set Port p1 tag=101
```


现在由于端口 p0 和 p1 属于不同的 VLAN，它们之间无法进行数据交换。我们使用 ovs-appctl ofproto/trace 生成一个从端口 p0 发送到端口 p1 的数据包，这个数据包不包含任何 VLAN tag，并观察 OVS 的处理过程

```
ovs-appctl ofproto/trace ovs-switch in_port=100,dl_src=d6:0f:7e:ed:11:e4,\
dl_dst=f2:0d:06:ff:79:d7 -generate
```
注意：上面第一个mac地址，是p0的，第二个mac地址是p1的，你需要替换，上面有获取mac地址的方法。

```
ovs-appctl ofproto/trace ovs-switch in_port=100,dl_src=d6:0f:7e:ed:11:e4,\
> dl_dst=f2:0d:06:ff:79:d7 -generate
Flow: metadata=0,in_port=100,vlan_tci=0x0000,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000
Rule: table=0 cookie=0 priority=1,in_port=100
OpenFlow actions=mod_nw_src:9.181.137.1,NORMAL
no learned MAC for destination, flooding

Final flow: unchanged
Relevant fields: skb_priority=0,in_port=100,vlan_tci=0x0000/0x1fff,dl_src=d6:0f:7e:ed:11:e4,
dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000,nw_src=0.0.0.0,nw_proto=0,nw_frag=no
Datapath actions: 1,4
```
在第一行输出中，“Flow:”之后的字段描述了输入的流的信息。由于我们没有指定太多信息，所以多数字段 （例如 dl_type 和 vlan_tci）被 OVS 设置为空值。

在第二行的输出中，“Rule:” 之后的字段描述了匹配成功的流表项。

在第三行的输出中，“OpenFlow actions”之后的字段描述了实际执行的操作。

最后一段以”Final flow”开始的字段是整个处理过程的总结，“Datapath actions: 4,1”代表数据包被发送到 datapath 的 4 和 1 号端口。

创建一条新的 Flow

对于从端口 p0 进入交换机的数据包，如果它不包含任何 VLAN tag，则自动为它添加 VLAN tag 101

```
ovs-ofctl add-flow ovs-switch "priority=3,in_port=100,dl_vlan=0xffff,\
actions=mod_vlan_vid:101,normal"
```


再次尝试从端口 p0 发送一个不包含任何 VLAN tag 的数据包，发现数据包进入端口 p0 之后, 会被加上 VLAN tag101, 同时转发到端口 p1 上

```
ovs-appctl ofproto/trace ovs-switch in_port=100,dl_src=d6:0f:7e:ed:11:e4,\
> dl_dst=f2:0d:06:ff:79:d7 -generate
Flow: metadata=0,in_port=100,vlan_tci=0x0000,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000
Rule: table=0 cookie=0 priority=1,in_port=100
OpenFlow actions=mod_nw_src:9.181.137.1,NORMAL
no learned MAC for destination, flooding

Final flow: unchanged
Relevant fields: skb_priority=0,in_port=100,vlan_tci=0x0000/0x1fff,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000,nw_src=0.0.0.0,nw_proto=0,nw_frag=no
Datapath actions: 1,4
root@ovs:~# ovs-ofctl add-flow ovs-switch "priority=3,in_port=100,dl_vlan=0xffff,\
> actions=mod_vlan_vid:101,normal"
root@ovs:~# ovs-appctl ofproto/trace ovs-switch in_port=100,dl_src=d6:0f:7e:ed:11:e4,\
> dl_dst=f2:0d:06:ff:79:d7 -generate
Flow: metadata=0,in_port=100,vlan_tci=0x0000,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000
Rule: table=0 cookie=0 priority=3,in_port=100,vlan_tci=0x0000
OpenFlow actions=mod_vlan_vid:101,NORMAL
no learned MAC for destination, flooding

Final flow: metadata=0,in_port=100,dl_vlan=101,dl_vlan_pcp=0,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000
Relevant fields: skb_priority=0,in_port=100,vlan_tci=0x0000,dl_src=d6:0f:7e:ed:11:e4,dl_dst=f2:0d:06:ff:79:d7,dl_type=0x0000,nw_proto=0,nw_frag=no
Datapath actions: push_vlan(vid=101,pcp=0),1,pop_vlan,3,push_vlan(vid=101,pcp=0),4
```


反过来从端口 p1 发送数据包，由于 p1 现在是带有 VLAN tag 101 的 Access 类型的端口，所以数据包进入端口 p1 之后，会被 OVS 添加 VLAN tag 101 并发送到端口 p0

```
ovs-appctl ofproto/trace ovs-switch in_port=101,dl_src=f2:0d:06:ff:79:d7,\
> dl_dst=d6:0f:7e:ed:11:e4 -generate
Flow: metadata=0,in_port=101,vlan_tci=0x0000,dl_src=f2:0d:06:ff:79:d7,dl_dst=d6:0f:7e:ed:11:e4,dl_type=0x0000
Rule: table=0 cookie=0 priority=0
OpenFlow actions=NORMAL
no learned MAC for destination, flooding

Final flow: unchanged
Relevant fields: skb_priority=0,in_port=101,vlan_tci=0x0000,dl_src=f2:0d:06:ff:79:d7,dl_dst=d6:0f:7e:ed:11:e4,dl_type=0x0000,nw_proto=0,nw_frag=no
Datapath actions: push_vlan(vid=101,pcp=0),1,2,4
```
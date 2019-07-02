---
title: ovs限流
date: 2019-07-02 16:30:40
mathjax: false
tags: [OpenFlow, OvS]
---


1. 网卡限速

```
ovs-vsctl set interface s1-eth1 ingress_policing_rate=5000
ovs-vsctl set interface s1-eth2 ingress_policing_rate=5000
```

2. 队列限流

```
在网卡s1-eth4上限制创建队列

ovs-vsctl set port s1-eth4 qos=@newqos — \
—id=@newqos create qos type=linux-htb queues=0=@q0 — \
—id=@q0 create queue other-config:max-rate=5000000
```

3. Meter表限速

```
设置交换机的工作模式和协议版本
ovs-vsctl set bridge s1 datapath_type=netdev
ovs-vsctl set bridge s1 protocols=OpenFlow13

下发meter表并查看
ovs-ofctl add-meter s1 meter=1,kbps,band=type=drop,rate=5000 -O OpenFlow13
ovs-ofctl show s1

下发流表并查看
ovs-ofctl add-flow s1 in_port=5,action=meter:1,output:6 -O openflow13
ovs-ofctl dump-flows s1 -O openflow13
```
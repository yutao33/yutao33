---
title: 100天阅读计划 -- 14
date: 2019-07-25 12:00:00
mathjax: true
tags: [Reading, Plan]
---


> Maglev: A Fast and Reliable Software Network Load Balancer

# Abstract

Maglev是google的一个网络负载均衡器

# Introduction

Maglev 2008年开始应用

# System Overview

## Frontend Serving Architecture

![](/images/Maglev-Fig2.png)

图2显示了google的前端服务架构，每个google服务有若干个VIP，Maglev 将每个VIP和服务endpoint沟通起来，并且通过BGP将IP通告给路由器；当用户访问google服务时，先请求DNS，得到最近的VIP；当router收到一个VIP数据包时，在集群里使用ECMP转发数据包到Maglev；当Maglev收到数据包，会选择一个服务endpoint，通过GRE隧道转发数据包，服务返回的数据包将直接封装为源地址为VIP，目的地址是请求的源地址。使用 Direct Server Return (DSR) 将响应直接返回到router。

## Maglev Configuration

![](/images/Maglev-Fig3.png)

每个Maglev包括一个controller和forwarder， controller周期检查forwarder的health状态，以决定发布/撤回BGP对VIP的宣告；forwarder里每个VIP对应若干个backend pool，每个backend pool可能递归的包括更多backend pool，每个backend pool 对应有一个或多个health check方法； forwarder的config manager负责解析的验证配置。

# Forwarder Design and Implementation

## Overall Structure
![](/images/Maglev-Fig4.png)

图4表示了Maglev forwarder的整体架构

The steering module performs 5-tuple hashing instead of round-robin scheduling for two reasons. First, it helps lower the probability of packet reordering within a connection caused by varying processing speed of different packet threads. Second, with connection tracking, the forwarder only needs to perform backend selection once for each connection, saving clock cycles and eliminating the possibility of differing backend selection results caused by race conditions with backend health updates. In the rare cases where a given receiving queue fills up, the steering module falls back to round-robin scheduling and spreads packets to other available queues. This fallback mechanism is especially effective at handling large floods of packets with the same 5-tuple.


## Fast Packet Processing

![](/images/Maglev-Fig5.png)

With proper support from the NIC hardware, we have developed a mechanism to move packets between the forwarder and the NIC without any involvement of the kernel, as shown in Figure 5.


## Backend Selection

Maglev 的connection tracking 使用固定大小的hash表将5-tuple hash值映射到backend

在分布式环境下存在问题：
1. 需要router把所有相同5元组的数据包都转发到相同的Maglev实例，但现实情况是Maglev的配置会变化 (?)
2. connection tracking 表大小有限，可能会在高负载或者SYN洪水攻击下被填满
所以需要使用一致性hash

## Consistent Hashing

参考 https://www.jianshu.com/p/9a9b269e68f7



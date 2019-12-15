---
title: Gossip协议
date: 2019-12-15 15:32:40
mathjax: true
tags: [Gossip, 分布式]
---

转载： http://kaiyuan.me/2015/07/08/Gossip/


Gossip是分布式系统中被广泛使用的协议，其主要用于实现分布式节点或者进程之间的信息交换。Gossip协议同时满足应用层多播协议所要求的低负载，高可靠和可扩展性的要求。由于其简单而易于实现，当前很多系统（例如Amazon S3，Usenet NNTP等）选择基于Gossip协议以实现应用层多播的功能。

## 什么是Gossip协议

Gossip Protocol利用一种随机的方式将信息散播到整个网络中。正如Gossip本身的含义一样，Gossip协议的工作流程即类似于绯闻的传播，或者流行病的传播。具体而言Gossip Protocol可以分为Push-based和Pull-based两种。Push-based Gossip Protocol的具体工作流程如下：

1. 网络中的某个节点随机的选择其他$b$个节点作为传输对象。
2. 该节点向其选中的$b$个节点传输相应的信息
3. 接收到信息的节点重复完成相同的工作

Pull-based Gossip Protol的协议过程刚好相反：

1. 某个节点$v$随机的选择$b$个节点询问有没有最新的信息
2. 收到请求的节点回复节点$v$其最近未收到的信息

当然，为了提高Gossip协议的性能，还有基于Push-Pull的混合协议。同时需要注意的是Gossip协议并不对网络的质量做出任何要求，也不需要loss-free的网络协议。Gossip协议一般基于UDP实现，其自身即在应用层保证了协议的robustness。

<!--more-->

## Gossip协议的性能

Gossip协议的分析是基于流行病学（Epidemiology）研究的。因此在分析Gossip的性能之前，需要首先介绍一下流行病学中基本的模型。

#### Epidemiology

流行病传染最基本的模型仅作如下几个假设：

1. $(n+1)$个人均匀的分布在一起
2. 每一对人群之间的传染概率是$\beta$，显然$0 \lt \beta \lt 1$.
3. 任意时刻，某个人要么处于infected的状态要么处于uninfected的状态.
4. 一旦某个人从uninfected状态转变成为infected状态，其一直停留在infected状态。

有了以上假设，我们可以进一步分析流行病的传染情况。我们记$t$时刻处于infected状态的人数为$y_t$，处于uninfected状态的人为$x_t$，那么初始状态 $y_0 = 1$, $x_0 = n$，并且在任何时候$x_t + y_t = n+1$.

考虑连续的时间，可知：

$$\frac{dx}{dt} = -{\beta}xy$$

解的：

$$x = \frac{n(n+1)}{n+e^{\beta{(n+1)}t}}$$

$$y = \frac{n+1}{1+ne^{-\beta{(n+1)}t}}$$

明显，当$t\to \infty$时，$x\to0,y\to(n+1)$，即经过足够的时间，所有的人都将被传染。

#### Gossip的性能

上述流行病传染模型为分析Gossip的性能提供了基础。在Gossip性能中，我们可以认为:
$\beta = b/n$（因为对每个节点而言，被其他节点选中的概率就是$b/n$)。我们令$t=clog(n)$，可以得到:

$$y \approx (n+1) - \frac{1}{n^{cb-2}}$$

这表明，仅需要$O(log(n))$个回合，gossip协议即可将信息传递到所有的节点。
根据分析可得，Gossip协议具有以下的特点:

1. 低延迟。仅仅需要$O(log(n))$个回合的传递时间。
2. 非常可靠。仅有$\frac{1}{n^{cb-2}}$个节点不会收到信息。
3. 轻量级。每个节点传送了$cblog(n)$次信息。

于此同事，Gossip协议的容错性比较高，例如，$50%$的丢包率等价于使用$b/2$带代替$b$进行分析；$50%$的节点错误等价于使用$n/2$来代替$n$，同时使用$b/2$来代替$b$进行分析，其分析结果不用带来数量级上的变化。
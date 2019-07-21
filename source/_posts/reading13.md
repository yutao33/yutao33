---
title: 100天阅读计划 -- 13
date: 2019-07-18 12:00:00
mathjax: true
tags: [Reading, Plan]
---


> Control Plane Compression

# ABSTRACT

We develop an algorithm capable of compressing large networks into smaller ones with similar control plane behavior: For every stable routing solution in the large, original network, there exists a corresponding solution in the compressed network, and vice versa.

# INTRODUCTION

In this paper, we tackle these problems by defining a new theory of control plane equivalence and using it to compress large, concrete networks into smaller, abstract networks with equivalent behavior. Because our compression techniques preserve many properties of the network control plane, including reachability, path length, and loop freedom, analysis tools of all kinds can operate (quickly) on the smaller networks, rather than their large concrete counterparts.

two contributions:
* A Theory of Control Plane Equivalence
* An Efficient Compression Algorithm

# OVERVIEW

## The Stable Routing Problem

An SRP instance assembles all of these ingredients:
1. a graph defining the network topology
2. a destination to which to route
3. a set of attributes
4. an attribute comparison relation
5. an attribute transfer function.

## Network Abstractions

A network abstraction defines precisely the relationship between the two. It is a pair of functions (f , h), where f is a topology abstraction that maps the nodes and edges of the concrete network to those of the abstract network, and h is an attribute abstraction that maps the concrete attributes in control plane messages to abstract ones.

![](/images/Bonsai-Fig1.png)

图一显示了一个CP-equivalent abstraction， 对于抽象和实际网络只有一个稳定解， $b_1$转发到$d$，对应的$\hat{b}$转发到$\hat{d}$.

## Effective Abstractions

![](/images/Bonsai-Fig2.png)


# STABLE ROUTING PROBLEM

TODO
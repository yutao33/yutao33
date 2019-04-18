---
title: 100天阅读计划 -- Day 6
date: 2019-04-16 15:13:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|15:13 | |

> Onix: A Distributed Control Platform for Large-scale Production Networks

# Abstract

- Onix, a platform on top of which a network control plane can be implemented as a distributed system. 
- Control planes written within Onix operate on a global view of the network, and use basic state distribution primitives provided by the platform. Thus Onix provides a general API for control plane implementations, while allowing them to make their own trade-offs among consistency, durability, and scalability.

# Introduction

requirements:
- Generality
- Scalability
- Reliability
- Simplicity
- Control plane performantce

contributions:
- First, Onix exposes a far more general API than previous systems
- Second, Onix provides flexible distribution primitives (such as DHT storage and group membership) allowing application designers to implement control applications without re-inventing distribution mechanisms

# Design

## Components

{% asset_img Figure1.png Figure1 %}

- Physical infrastructure
- Connectivity infrastructure
- Onix
- Control logic

## The Onix API

- Onix’s API consists of a data model that represents the network infrastructure, with each network element corresponding to one or more data objects.
- The copy of the network state tracked by Onix is stored in a data structure we call the Network Information Base (NIB)
- While Onix handles the replication and distribution of NIB data, it relies on application-specific logic to both detect and provide conflict resolution of network state as it is exchanged between Onix instances, as well as between an Onix instance and a network element.

## Network Information Base Details

{% asset_img Figure2_Table1.png Figure2_Table1 %}



# Scaling and Reliability

## Scalability
Onix supports three strategies that can used to improve scaling.
- Partitioning: a particular controller instance keeps only a subset of the NIB in memory and up-to-date
- Aggregation: one instance of Onix can expose a subset of the elements in its NIB as an aggregate element to another Onix instance.
- Consistency and durability: The control logic dictates the consistency requirements for the network state it manages.

## Reliability

Control applications on Onix must handle four types of network failures: 
- forwarding element failures, 
- link failures, 
- Onix instance failures, 
- failures in connectivity between network elements and Onix instances (and between the Onix instances themselves






# Distributing the NIB

## Overview

## State Distribution Between Onix Instances

For network state needing high update rates and availability,
Onix provides a one-hop, eventually-consistent,
memory-only DHT 


--------

{% raw %}
<iframe src="//player.bilibili.com/player.html?aid=46685811&cid=81778658&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
{% endraw %}


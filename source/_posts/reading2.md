---
title: 100天阅读计划 -- Day 2
date: 2019-04-05 14:17:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|14:17 | 1 day delay|

> VL2: A Scalable and Flexible Data Center Network

# Abstract

a practical network architecture that scales to support huge data centers with uniform high capacity between servers, performance isolation between services, and Ethernet layer-2 semantics.

- flat addressing
- Valiant Load Balancing
- end-system based address resolution to scale to large server pools

# INTRODUCTION

data centers mush achieve high utilization, and key to this is the property of agility -- the capacity to assign any server to any service.

three objectives.
- Uniform high capacity
- Performance isolation
- Layer-2 semantics

VL2 consists of a network built from low-cost switch ASICs arranged into a Clos topology [7] that provides extensive path diversity between servers. Our measurements show data centers have tremendous volatility in their workload, their traffic, and their failure patterns. To cope with this volatility, we adopt Valiant Load Balancing (VLB) [18, 23] to spread traffic across all available paths without any centralized coordination or traffic engineering.

When a server sends a packet, the shim-layer on the server invokes a directory systemto learn the actual location of the destination and then tunnels the original packet there. the shim-layer also helps eliminate the scalability problems created by ARP in layer-2 networks, and the tunneling improves our ability to implement VLB. these aspects of the design enable VL2 to provide layer-2 semantics, while eliminating the fragmentation and waste of server pool capacity that the binding between addresses and locations causes in the existing architecture.

contributions:
- find there is tremendous volatility in the traffic
- design, build, and deploy VL2
- apply Valiant Load Balancing in a new context, the inter-switch fabric of a data center
- justify the design trade-offs

# BACKGROUND

to isolate different services or logical server groups (e.g., email, search, web front ends, web back ends), servers are partitioned into virtual LANs (VLANs). Unfortunately, this conventional design suffer from three fundamental limitations:

- Limited server-to-server capacity
- Fragmentation of resources
- Poor reliability and utilization

# MEASUREMENTS & IMPLICATIONS

developing the mechanisms on which to build the network requires a quantitative understanding of the traffic matrix (who sends how much data to whom and when?) and churn (how o ften does the state of the network change due to changes in demand or switch/link failures and recoveries, etc.?).

- Data-Center Traffic Analysis
    - Analysis of Netflow and SNMP data
- Flow Distribution Analysis
    - Distribution of flow sizes
    - Number of Concurrent Flows
- Traffic Matrix Analysis
    - Poor summarizability of traffic patterns
    - Instability of traffic patterns
- Failure Characteristics


{% asset_img Figure2.png Figure2 %}
{% asset_img Figure3.png Figure3 %}
{% asset_img Figure4.png Figure4 %}


# VIRTUAL LAYER TWO NETWORKING

briefly discuss our design principles and preview how they will be used in the VL2 design

- Randomizing to Cope with Volatility  (VLB)
- Building on proven networking technology
- Separating names from locators
- Embracing End Systems

## Scale-out Topologies


## VL2 Addressing and Routing

### Address resolution and packet forwarding
{% asset_img Figure6.png Figure6 %}

- The network infrastructure operates using location-specic IP addresses (LAs); all switches and interfaces are assigned LAs, and switches run an IP-based (layer-3) link-state routing protocol that disseminates only these LAs. This allows switches to obtain the complete switch-level topology, as well as forward packets encapsulated with LAs along shortest paths. 
- applications use application-specic IP addresses (AAs), which remain unaltered no matter how servers’ locations change due to virtual-machine migration or re-provisioning. Each AA (server) is associated with an LA, the identier of the ToR switch to which the server is connected. 
- The VL2 directory system stores the mapping of AAs to LAs, and this mapping is created when application servers are provisioned to a service and assigned AA addresses.

### Random traffic spreading over multiple paths

Figure 6 illustrates how the VL2 agent uses encapsulation to implement VLB by sending traffic through a randomly-chosen Intermediate switch. The packet is first delivered to one of the Intermediate switches, decapsulated by the switch, delivered to the ToR’s LA, decapsulated again, and finally sent to the destination.



### Backwards Compatibility

- Interaction with hosts in the Internet
- Handling Broadcast

## Maintaing Host Information using the VL2 Directory System

The VL2 directory provides three key functions: (1) lookups and (2) updates for AA-to-LA mappings; and (3) a reactive cache update mechanism so that latency-sensitive updates (e.g., updating the AA to LA mapping for a virtual machine undergoing live migration) happen quickly.


# EVALUATION

- VL2 Provides Uniform High Capacity
- VL2 Provides VLB Fairness
- VL2 Provides Performance Isolation
- VL2 Convergence After Link Failures
- Directory-system performance


--------

{% raw %}
<iframe src="//player.bilibili.com/player.html?aid=3905462&cid=6319180&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
{% endraw %}


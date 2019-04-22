---
title: 100天阅读计划 -- Day 7
date: 2019-04-21 15:50:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|15:50 | |

> A General Approach to Network Configuration Verification

# ABSTRACT

We present Minesweeper, a tool to verify that a network satisfies a wide range of intended properties such as 
- reachability or 
- isolation among nodes, 
- waypointing,
- black holes,
- bounded path length,
- load-balancing,
- functional equivalence of two routers, and
- fault-tolerance

# INTRODUCTION

{% asset_img Figure1.png Figure1 %}

The main challenge in developing Minesweeper was scaling such a general tool. We addressed it by combining the following ideas from networking and verification literature:

- Graphs (not paths)
- Combinational search (not message set computation)
- Stable paths problem
- Slicing and hoisting optimizations


# MOTIVATION

## Paths vs. graphs

## Message sets vs. combinational search



# THE BASIC NETWORK MODEL

- Modeling data plane packets
- Modeling protocol interactions
- Encoding control plane information
- Encoding import filters
- Encoding route selection
- Encoding route export
- Encoding data plane constraints
- Encoding properties







# GENERALIZING THE MODEL
This section describes how we encode several additional features of network configurations.

- Link-state protocols
- Distance-vector protocols
- Static routes
- Aggregation
- Multipath routing
- BGP communities
- iBGP
- Route reflectors
- Multi-exit discriminator (MED)
- Design Decisions and Limitations




# PROPERTIES
As noted earlier, our model allows us to express a range of properties using SMT constraints.We now show how to encode some common properties of interest.




# OPTIMIZATIONS




# IMPLEMENTATION
- Batfish (parse vendor-specific configurations)
- Z3 SMT solver


# EVALUATION

--------

{% raw %}

{% endraw %}


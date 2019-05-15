---
title: 100天阅读计划 -- 8
date: 2019-04-23 14:15:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|14:15 |  -   |

> p4v: Practical Verification for Programmable Data Planes



# ABSTRACT

a practical tool for verifying data planes described using the P4 programming language
- a novel mechanism for incorporating assumptions about the control plane
- domain-specific optimizations which are needed to scale to large programs

with just a few hundred lines of control-plane annotations, p4v is able to verify critical safety properties for `switch.p4`



# INTRODUCTION

- Our vision of verified data planes is inspired in part by recent successes in the formal methods community
- SAT and SMT solvers, which underpin many automated tools, have become very fast in recent years and are now able to scale to extremely large problem instances in many common cases

Challenges:
- One issue is that a P4 program is really only half of a program. The contents of the match-action tables are not known until they are populated by the control plane at run time
- Another challenge concerns scalability. Although P4 programs are limited to simple data structures and control flow, practical programs can be quite large, often running to tens of thousands of lines of code



# BACKGROUND ON P4

- headers
- parsers
- tables
- actions
- controls



# DATA-PLANE PROPERTIES

- General safety properties.
    - reading invalid headers
    - ensuring that header stacks are only ever accessed within statically declared bounds, 
    - that arithmetic operations do not overflow, 
    - and that the compiler-generated deparser emits all headers that are valid at the end of the egress pipeline.
    - {% asset_img 3_1.png 3_1 %}
- Architectural properties
    - it is easy to make mistakes such as specifying conflicting forwarding operations (e.g., drop and multicast), 
    - specifying operations that are unimplementable (e.g., recirculating the packet an excessive number of times), 
    - or forgetting to make a forwarding decision at all, letting the target decide what to do with the packet.
    - {% asset_img 3_2.png 3_2 %}
- Program-specific properties
    - For example, the designer of a switch might want to check that broadcast traffic is handled correctly, 
    - while the designer of a router might want to check that the IPv4 ttl field is correctly decremented on every packet. 
    - Or, in the firewall example, as discussed previously, we might want to prove that a given internal server is isolated from the rest of the network.
    - {% asset_img 3_3.png 3_3 %}




# VERIFICATION METHODOLOGY

1. we first build a first-order formula that captures the execution of the program in logic, leveraging the fact that P4 programs denote functions on finite sequences of bits (i.e., packets) parameterized on finite state (i.e., switch registers), 
2. and then use an automated theorem prover to check whether there exists an initial state that leads to a violation of one or more correctness properties.


- One of the key challenges we faced in building p4v is that the P4 language lacks a formal semantics
- To address this challenge, we defined a translation from P4 programs to Guarded Command Language (GCL), an imperative language with non-deterministic choice (see Figure 2) [11].
- {% asset_img 4_1.png 4_1 %}
- we translate each table application into a non-deterministic choice between the actions declared for the table and, if the table doesn’t declare a default action, a `no-op` action for when the table misses. For example, the translation of apply(acl) is the following:
    `assume(true) [] allow() [] deny()`
- Figure 3 gives the formal definition of a function `wlp` (for weakest liberal preconditions [11]) that computes verification conditions for a GCL command c and postcondition Q, which is initially true. Most of the cases are intuitive:
    - assignment substitutes the expression for the variable in the postcondition,
    - sequential composition threads the postcondition through c2 then c1, 
    - and non-deterministic choice computes the conjunction of the weakest preconditions for c1 and c2.
    - The cases for assumptions and assertions handle annotations used in program-specific properties such as the firewall example. 
    - Assumptions produce an implication from the assumed formula to the postcondition, 
    - while assertions conjoin the asserted formula with the postcondition. 
    - The verification conditions for a P4 program `p` are given by `wlp(c, true)`, where `c` is the translation of `p` into GCL.
- {% asset_img 4_2.png 4_2 %}




# CONTROL-PLANE INTERFACE

we constrain the behavior of the control plane using symbolic constraints in a control-plane interface. Figure 4 defines syntax for the additional expressions that can be used to define the controlplane interface. 
- The expression `reach(t)` is set to 1 if the execution reaches an application of t . 
- The expression `reads(t, k)` is set to the data-plane value read by `t` identified by `k`.
- Similarly, `wildcard(t, k)` evaluates to 1 if the value identified by `k` is matched against an all-wildcard pattern.
- The expressions `hit(t)` and `miss(t)` evaluate to 1 if executing the table hits and misses respectively. 
- Finally, the expression `action_data(t, a, x)` returns the value of the action data for parameter `x` in action `a` of table `t`.

{% asset_img 5_1.png 5_1 %}

the highlevel idea is as follows: we instrument the program with ghost variables to keep track of which tables and actions are executed, we translate the control-plane interface into a logical formula involving those ghost variables, and finally we predicate every assertion in the program on this formula.


# IMPLEMENTATION

{% asset_img 6_1.png 6_1 %}

- Parsing and type checking
- Instrumentation
- Inlining
- Annotation
- Passivization
- Optimizations
- Verification conditions
- Counter-example generation

# CASE STUDIES

- Header validity for switch.p4
- NetCache parser roundtripping
- NetPaxos bug
- Enabling compiler optimizations

# EVALUATION



--------

{% raw %}
<iframe src="//player.bilibili.com/player.html?aid=49803272&cid=87197612&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
{% endraw %}


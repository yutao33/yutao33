---
title: 100天阅读计划 -- Day 1
date: 2019-04-04 11:48:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|13:23 | 16:02|

> Packet Transactions: High-Level Programming for Line-Rate Switches


# ABSTRACT

packet transaction: a sequential packet-processing code block that is atomic andisolated from other such code blocks.

# INTRODUCTION

Contributions
- Banzai
- Domino
- compiler from Domino packet transactions to a Banzai target

# A MACHINE MODEL FOR LINERATE SWITCHES

- Barefoot Networks’ Tofino
- Intel’s FlexPipe
- Cavium’s XPliant Packet Architecture

{% asset_img BanzaiModel.png BanzaiModel %}

- Banzai is a feed-forward pipelien consisting of a number of stages executing synchronously on every clock cycle; 
- Banzai's pipeline is deterministic, never stalls, and always sustains line rate.
- Atoms : Banzai's processing units
- An atom is represented by a body of sequential code that captures the atom’s behavior. It may also contain internal state local to the atom. An atom completes execution of this entire body of code, modifying a packet and any internal state before processing the next packet.

# PACKET TRANSACTIONS

{% asset_img DominoExample.png DominoExample %}

Domino’s syntax (Figure 4) is similar to C, but with several constraints (Table 1). These constraints are required for deterministic performance. Memory allocation, unbounded iteration counts, and unstructured control flow cause variable performance, which may prevent an algorithm from achieving line rate. Additionally, within a Domino transaction, each array can only be accessed using a single packet field, and repeated accesses to the same array are allowed only if that packet field is unmodified between accesses.

# THE DOMINO COMPILER

{% asset_img DominoCompilerPasses.png DominoCompilerPasses %}

- all-or-nothing model
- Preprocessing
    - Branch removal
    - Rewriting state variable operations
    - Converting to static single-assignment form
    - Flattening to three-address code
- Pipelining
    - codelets, where each codelet is a sequential block of three-address code statements.
    - Pipelined Virtual Switch Machine
- Code generation
    - Resource limits & Computational limits
    - use the SKETCH program synthesizer

# EVALUATION

{% asset_img Table3.png Table3 %}




--------------


{% iframe https://player.bilibili.com/player.html?aid=43426592&cid=76113255&page=1 [640] [430] %}




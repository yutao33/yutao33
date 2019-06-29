---
title: 100天阅读计划 -- 12
date: 2019-06-29 12:00:00
mathjax: true
tags: [Reading, Plan]
---


> The Case for an Intermediate Representation for Programmable Data Planes (SOSR15)

# Abstract

NetASM 是一种设备无关的低级汇编语言，允许上层编译器优化数据转发流水线性能

# Introduction

设备的可编程性向细粒度灵活发展，这些设备依然很难编程，不同高级语言的使用使得很难直接编译到一个设备，所以需要一个中间语言，这篇paper引入了NetASM，即是作为一种中间表达

作为中间语言需要具备，1. 表达性强，可以表示大量底层设备的特性，2. 允许上层编译器利用现有优化框架

# NetASM: An Intermediate Representation

NetASM引入了持久状态和单packet状态，顺序和并行执行

## Persistent State and Per-Packet State

## Sequential and Concurrent Execution

SEQ 顺序
CNC 全并行
ATM 原子执行

## NetASM Instruction Set

The NetASM instruction set has only 23 instructions, as shown in Table 1.

![](/images/NetASM-Table1.png)

# Optimizations

1. Data-Flow Analysis Framework
    We implemented two new kinds of data-flow analysis techniques, called field-reachability and field-usability analysis,
2. Dead-Code Elimination
3. Dead-Store Elimination
4. Code Motion
    移动指令

# Preliminary Evaluation

## Cost Model

We evaluate NetASM based on an abstract cost model that we develop;
we use this model to calculate the abstract area and latency
of a NetASM program. The values of area and latency simply have
weights that reflect the relative costs of each instruction; higher
area and latency costs reflect more area usage and higher latency,
respectively.

## How Well Do the OptimizationsWork?

We evaluated the application against four different combinations of
optimizations and measured the improvement in area and latency
with respect to the baseline values of the original program.



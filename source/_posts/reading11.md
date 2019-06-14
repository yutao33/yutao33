---
title: 100天阅读计划 -- 11
date: 2019-06-16 13:54:00
mathjax: true
tags: [Reading, Plan]
---


> Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center


# Abstract


# Introduction

In this paper, we propose Mesos, a thin resource sharing layer that enables fine-grained sharing across diverse cluster computing frameworks, by giving frameworks a common interface for accessing cluster resources.

# Target Environment

![](/images/Mesos-Fig1.png)

Most jobs are short (the median job being 84s long), and the jobs are composed of fine-grained map and reduce tasks (the median task being 23s), as shown in Figure 1.

Mesos目标在于对于不同的集群计算框架提供细粒度的资源共享

# Architecture

## Design Philosophy

因为集群框架也在快速的发展，Mesos的理念是设计最少的接口实现高效的资源共享

## Overview

![](/images/Mesos-Fig2.png)

The master implements fine-grained sharing across frameworks using resource offers. Each resource offer is a list of free resources on multiple slaves. The master decides how many resources to offer to each framework according to an organizational policy, such as fair sharing

Each framework running on Mesos consists of two components: a scheduler that registers with the master to be offered resources, and an executor process that is launched on slave nodes to run the framework tasks.

master决定哪些资源提供给framework, 由framework的scheduler选择在哪些资源上运行任务，当一个framework接受这些资源，会向Mesos发送要运行的任务的描述，framework可以拒绝使用这些资源，为了避免频繁的拒绝行为，framework可以设置一个filter限制可以使用的资源

## Resource Allocation

Mesos的分配决策算法，采用了插件的方式，论文里实现了多资源的max-min fairness 和绝对优先级的分配

在某些任务长期运行时，Mesos可以通知framework暂停这些任务，framework需要更多资源时也可以向Mesos提出请求重新分配

## Isolation

目前Mesos使用Container技术实现资源隔离

## Making Resource Offers Scalable and Robust

如果framework长时间没有相应Mesos提供的资源，Mesos会撤回这些请求

## Fault Tolerance

1. master被设计为soft state  (stateless protocol?)的，master需要的状态有从节点列表，运行的framework列表和运行的task列表，这些状态就足够计算每个framework占用的资源以及运行分配算法
2. 如果从节点故障，Mesos向framework报告，由framework处理
3. Mesos允许framework注册多个schudular

## API Summary

![](/images/Mesos-Table1.png)


# Mesos Behavior

## Definitions, Metrics and Assumptions

## Homogeneous Tasks

## Placement Preferences

## Heterogeneous Tasks

## Framework Incentives

## Limitations of Distributed Scheduling


# Implementation

1. Hadoop Port
2. Torque and MPI Ports
3. Spark Framework


# Evaluation

1. Macrobenchmark
2. Overhead
3. Data Locality through Delay Scheduling
4. Spark Framework
5. Mesos Scalability
6. Failure Recovery
7. Performance Isolation

# Related Work

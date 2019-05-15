---
title: 100天阅读计划 -- 10
date: 2019-05-15 10:54:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|11:00 | -    |

> Distributed Network Monitoring and Debugging with SwitchPointer (NSDI18)


# Abstract

The key contribution of SwitchPointer is to efficientl provide network visibility by using switch memory as a "directory service" each switch, rather than storing the data necessary for monitoring functionalities, stores pointers to end-hosts where relevant telemetry data is stored.

# Introduction

# Motivation

- Too much traffic
- Too many red lights
- Traffic cascades


# SwitchPointer Overview

- SwitchPointer Switches
    - embedding the telemetry data into packet header;
    - maintaining pointers to end hosts where the telemetry data for packets processed by the switch are stored;
    - coordinating with an analyzer for monitoring and de bugging network problems.
- SwitchPointer End-hosts
- SwitchPointer Analyzer



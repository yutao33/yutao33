---
title: 100天阅读计划 -- Day 4
date: 2019-04-08 14:26:00
mathjax: true
tags: [Reading, Plan]
---


|start | end  |
|----  | -----|
|14:26 | 16:18|

> P5: Policy-driven optimization of P4 pipeline


# Abstract

a system that exploits knowledge of application deployments embedded in a high-level policy abstraction to: 
- detect features that are used by applications in a mutually-exclusive way and thereby remove inter-feature dependencies between the tables implementing these features in a network switch.
- detect and remove the features that are not used by any application/traffic on the switch in a given topology. 


# INTRODUCTION

- not all traffic will require all features
- not all features need to be enabled in every switch
- pipeline concurrency

Contributions
- Using high level policies, we create tenant-to-feature mappings, to correctly identify dependencies between the tables, thus creating a more efficient pipeline switch.
- We further decide which specific features must be supported in the network topology to meet the networkwide policies, and thus enable only those features in selective P4 switches.
- We demonstrate up to 50% improvement in pipeline efficiency, in terms of number of pipeline stages, with real P4 switch programs.

# BACKGROUND

## Policy Intent

- policy intents can be defined with a group of end points (EPG) and their associated rules
- Systems like PGA (Policy Graph Abstraction) use labels and label-trees to specify high level policy intents
- In this paper, we use the tenant-to-feature mappings from the label tree to identify mutually exclusive features. By mutually-exclusive features (F1, F2), we mean that a packet which requires feature F1 will not require feature F2 and vice versa.

## P4 Language

A typical P4 program implementing various switch features has more tables than the number of physical stages; Many of the logical tables can be concurrently executed in the same stage as long as they don’t have any dependencies on one another and the stage has enough resources to run the tables.

In this paper, we propose to identify and remove unnecessary table dependencies by exploiting knowledge of application requirements and deployments embedded in high-level policy abstractions.

# GENERATING ACCURATE TABLE DEPENDENCIES

{% asset_img Figure1.png Figure1 %}
{% asset_img Table1Table2.png Table1Table2 %}

As shown in Table 2, the original P4 compiler would have put all these tables in a separate pipeline stage (five pipeline stages). This happens because according to the P4 compiler, all of the tables have a dependency relationship with every other table as they are all using the destination IP address. On the other hand, the label tree and tenant-to-feature mapping tells us that since all three features are mutually exclusive, there is no inter-feature table dependency and hence the number of pipeline stages can be reduced to three. The two other stages are freed up and can be used to either host new advanced features or increase the size of the standard features placed in stages 1-3.

# FEATURE REMOVAL

{% asset_img Figure3_4_5.png Figure3_4_5 %}

Figure 3 represents a complex egress pipeline with 8 network features enabled. The total number of stages in this pipeline is 10. In Figure 3, shaded blocks represent the stages occupied by each feature and the arrows represent the dependency between blocks. One of the main things that stands out is the interdependencies between features. An obvious way to reduce the number of pipeline stages is to reduce the number of features to only the ones required, which is shown in Figure 4. This reduces the number of stages not just because the P4-tables associated with the removed features are missing, but also because the dependencies to those features are avoided and hence they can be placed in an earlier stage.

Upgrading the software stack takes up to a few seconds while the data plane update is almost instantaneous, orders of magnitude faster.

# EVALUATION

{% asset_img Figure6_7_8.png Figure6_7_8 %}


--------

{% raw %}
<iframe src="//player.bilibili.com/player.html?aid=8506617&cid=14004931&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
{% endraw %}


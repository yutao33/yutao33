---
title: 'Experiment: OpenWhisk'
date: 2018-08-07 16:08:41
tags: Experiment
---

# Serverless

Refer to [Apache OpenWhisk by Animesh Singh and John Zaccone](https://www.youtube.com/watch?v=phsSvI7JB48)

## What is Serverless good for?

Serverless is good for

* short-running
* stateless
* event-driven

example:

* Microservices
* Mobile Backends
* Bots, ML Inferencing
* IoT
* Data (Stream) Processing
* Cognitive
* Scatter/Gather workloads

Serverless is not good for

* long-running
* stateful
* number crunching

example:

* Databases
* Deep Learning Training
* Heavy-Duty Stream Analytics
* Spark/Hadoop Analytics
* Numerical Simulation
* Video Streaming


## Introducing OpenWhisk, a fabric and platform for the serverless, event-driven programming model.

## Programming model

* Trigger: A class of events that can happen
* Actions: An event-handler. Can be chained to create sequences to increase flexibility and foster reuse.
* Rules: An association of a trigger and an action
* Packages: A shared collection of triggers and actions


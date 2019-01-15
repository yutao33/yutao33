---
title: Turing machine simulator (C)
date: 2019-01-15 09:17:40
mathjax: true
tags: [Turing machine, C]
---


refer to https://web.archive.org/web/20150909215048/http://en.literateprograms.org:80/Turing_machine_simulator_(C)#chunk%20def:state%20tracing

single-tape Turing machine $M = \left(Q, \Gamma, q_{0}, \sqcup, F, \delta \right)$.

* $Q$ is a finite set of states;
* $\Gamma$ is the finite tape alphabet (the symbols that can occur on the tape);
* $q_{0}\in Q$ is the initial state;
* $\sqcup \in \Gamma$ is the blank symbol;
* $\Gamma \subset Q$ is the set of accepting (final) states;
* $\delta: Q \times \Gamma \rightarrow Q \times \Gamma \times \\{L, R\\}$ is the transition function which determines the action performed at each step.

Initially the tape has the input string on it, followed by an infinite number of blank symbols ($\sqcup$), and the head is at the left end of the tape. At each step we use the transition function to determine the next state, the symbol written on the tape just prior to moving, and the direction to move the head, left (L) or right (R). If we ever reach a final state, the machine halts.

{% include_code TuringMachine lang:c TuringMachineCImpl.c %}



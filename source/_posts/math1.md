---
title: 证明（0，1）是不可列集
date: 2018-10-14 16:13:40
mathjax: true
tags: [Math, 泛函分析]
---


考虑区间 $(0,1\]$, 若$|(0,1]| = \aleph$ (集合的势是可列集), 则存在一一满映射$f: \mathbb{N} \rightarrow (0,1\]$. 每个$f(n)$ 可唯一的表示为十进制小数:

\begin{equation}
f(n)=0.a_{n_1} a_{n_2} \cdots a_{n_k} \cdots, n=1,2,\cdots.
\end{equation}

若$a_{n_n}=1$，取$b_n=2$，若$a_{n_n} \neq 1$，取$b_n=1$($n=1,2,\cdots$)。记$x=0.b_1 b_2 \cdots b_n \cdots$，则$x \in (0,1\]$，于是存在$n\in \mathbb{N}$，使得$x=f(x)$。 即

\begin{equation}
0.b_1 b_2 \cdots b_n = 0.a_{n_1} a_{n_2} \cdots a_{n_k} \cdots .
\end{equation}

由此推出$b_n = a_{n_n}$，得出矛盾。因此区间 $(0,1\]$ 不是可列集。


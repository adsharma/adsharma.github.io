---
layout: post
title: LLMs and Databases
date: 2026-04-13 00:00:00 +0000
---

In his [blog post](https://shah.posthaven.com/llms-are-compute-context-is-memory-inference-speed-is-king), Mehul Shah asks:

> So, what does this (LLMS) all mean for infrastructure like databases? I argue it has three implications:
> First, the most prosaic implication is that we'll need to build databases **_for_** LLMs. (Higher throughput)
> The second and more interesting implication is that we will likely build databases **_with_** LLMs. (Rewrite/Custom Develop database code itself)
> Finally, we will build all of our analytics engines (i.e. data warehouses) **_on_** LLMs and unstructured data will be first-class.

Here we argue that this is a LLM centric view with database as a junior partner trying to adjust itself to the changing reality. we will then present a database maximalist view and then outline ways in which LLMs and Databases can work together as equal partners to solve business problems.

## Database Maximalism

LLMs are themselves a probabilistic graph databases that won the hardware lottery. That's a minor variation what Chaitanya Joshi argues [here](https://arxiv.org/abs/2506.22084).

Pre-training can be seen as the process of creating a clustered index on the graph followed by a lossy compression of all leaf nodes, leaving only the index around. Key observation: original data is no longer is recoverable because of the lossy compression.

![Clustered Index](/assets/img/2026-04-13-llms-databases-clustered-index.png)

## LLMs and Databases as equal partners

A healthier relationship is possible between these two fields. One is an established science and a major chunk of corporate spending on tech. The other is an upcoming star that is going to change the tech industry and everything adjacent to it.

A few ideas:

* LLMs are good at recognizing patterns. Use them to organize storage. Instead of throwing rows in a heap and building an index, organize storage based on what LLMs understand about it.
* LLMs as query planners - apart from the stats computed in the local db, we can bring world knowledge to pick query plans.
* LLMs and Databases can collaborate to bridge the probabilistic and deterministic, seamlessly switching between the two. One nice side effect of this is that RBAC (role based access control) becomes possible.

Other questions to ponder: the meaning of pre-training and continuous learning changes significantly with Database tech taking a more significant role vs today.

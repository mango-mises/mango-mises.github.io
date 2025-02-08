---
layout: post
title: Scalable Integrity Webs
tags: [consensus, proof, scale, integrity]
comments: true
mathjax: true
author: IV
---

Our goal is to build a scalable interity web. As remarked in a [previous post](https://unthoughts.github.io/2025-01-18-integrity-matters/), network integrity is facilitated at scale by state of the art consensus protocols (necessarily DAG-based), and social integrity is achieved by making careful use of the partial information ensured by wide DAGs. The remaining challenge is to attain computational integrity at scale. To this end we have the ideal tool: proofs. Recycling my favorite quote from [Checking Computations in Polylogarithmic Time](https://dl.acm.org/doi/pdf/10.1145/103418.103428):

> In this setup, a single reliable PC can monitor the operation of a herd of supercomputers working with possibly extremely powerful but unreliable software and untested hardware.

In my opinion the key is an architecture built around deferred computation. We obviously want to defer computation away from observers. However, since operator decentralization is also crucial for network integrity, we should also defer computation away from operators.

## State/logic zones

## Sequencing, execution, statements

Todos:

1. compute and memory/storage
2. Consensus on execution is a middle-ground: it gives a concrete statement (e.g. state-diffs) for consumption to those who trust consensus.
3. Ideally every execution is a one-time event, without everyone else verifying.
4. Compare consensus on ordering vs consensus on ordering & verification.

## Against the conservative approach to opcodes/syscalls

Cheap and fast verification is important since we must invite deferred computation!

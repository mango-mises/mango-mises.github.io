---
layout: post
title: Thoughts on distributed ledger design
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [distributed ledger, consensus, proof, DAG, architecture, scale, integrity]
author: IV
---

## Integrity matters

Distributed ledgers face three integrity problems:
1. Network integrity
2. Social integrity
3. Computational integrity

### Network integrity

Network integrity is provided by consensus protocols. In my view, they transform common belief[^1] in some honest threshold of network nodes[^2] into common belief in consistency of views of the ledger by all observers. I think there is formal content to this viewpoint. For instance, in the oral message algorithm from the [Byzantine Generals Problem](https://dl.acm.org/doi/10.1145/357172.357176) paper, it's cool to interpret a chain of messages (a path in each agent's local tree) as a sequence of belief operators. Disregarding introspections, this seems to lead to "practical" common belief.

Network integrity requires a large and cohesive network. The moral solution is thus a single consensus layer, without fragmentation and without duplicate efforts. Emphatically, the network comprising this layer consists not only of "operator" nodes, but also of "relayer" nodes that participate in P2P efforts and contribute to cohesion.

Such a vision quickly runs into a scaling question: Is there tension between security, performance, and network size?

#### DAGs

Causality relations are canonically structured as DAGs:
1. Events may be causally independent.[^5] These are path-disconnected nodes.
2. An event may causally depend on multiple independent events. This is a node with several parents.
3. Two events cannot causally depend on each other.

Let's fix a context. The details don't really matter yet, but in ours, events are blocks. Each view of the world has its own DAG, encoding known information. Since causality is absolute, all such DAGs are consistent in the sense that their superposition is a also a DAG, corresponding to the union of information from consituent views.  Note the stark contrast to possible of conflicting views of e.g. the longest/heaviest chain. In my opinion, such discrepancies hint that ordering must be founded on full information, i.e the entire ambient DAG as opposed to arbitrarily chosen substructures.

Since causality relations between events/actions are naturally structured as DAGs, there's no inherent argument in favor of sequential monopolies. In our distributed ledger context, there's a very compelling argument _against_ sequential monopolies - they are hugely centralized by definition. In the next section we'll distill this principled intuition into strong arguments in favor of _wide DAGs_ for the sake of social integrity.

#### PoS quorums

Monolithic PoS quorum-based consensus protocols periodically gather many operators in a dedicated voting room[^3] until a quorum of stake is reached. Large quorums become bottlenecks due to communication overhead. Small quorums forgo security. These extremes are interpolated via randomly selected committees: small quorums are accumulated until a global stake threshold is reached, granting finality. [TODO: refer to Vitalik's discussion about committees]

I want to make the security point explicit:
1. In monolithic PoS quorum-based protocols, a small quorum means consensus is secured by a small portion of stake.
2. In PoW, consensus is secured by all of the hashrate[^4].

Monolithic PoS quorum-based protocols feature a common pattern which obstructs their throughput by strongly coupling between consensus and dissemination of new transactions.
1. A block authored by Alice is propagated and individually voted on by various nodes.
2. The individual votes are collectively consumed (collected into a quorum), either by Alice or by the next leader Bob.

In my opinion it's useful to think of these as events. Each individual voting event only depends on the block authored by Alice. However, the quorum event causally depends on multiple (often independent) voting events. Evidently, a chain structure precludes quorum events at the protocol-level. Thus monolithic protocols _must_ defer quorums to meta-events. DAGs on the other hand are perfectly suited for this natural viewpoint on quorums. The DAG approach also allows for natural removal of consensus from the critical path of new transaction dissemination and vice versa, as remarked in [Narwhal and Tusk](https://arxiv.org/pdf/2105.11827).

I _guessing_ the DAG paradigm won't improve the _security_ of PoS quorum-based protocols because it hits the same underlying bottleneck, only now it's explicit as the maximal sustainable DAG width. Specifically, I'm guessing the width can't exceed several hundreds of blocks, which is similar to the Tendermint validator set bounds I've heard from the folks at Informal Systems.

#### PoW

In my opinion the field of PoW consensus is essentially complete with [DAG KNIGHT](https://eprint.iacr.org/2022/1494.pdf). This amazing paper is the culmination of an admirable intellectual journey going back at least as far as [2015](https://eprint.iacr.org/2013/881.pdf). I'm very lucky to have access to both Yoni and Sutton, and I'll try to write about DAG KNIGHT in the near future.

### Social integrity

Operators... fee market, MEV, Oracles

Price discovery when you have access to different suppliers

Wide DAGs

### Computational integrity

## Sequencing, execution, statements
s

[^1]: Common belief in the context of modal logic.
[^2]: It is customary to disregard the distinction between network operators (creators of blocks/votes) and other nodes that can only manipulate the propagation of messages. In practice this distinction is important.
[^3]: Protocols often use two voting rounds to finalize. The second round ensures safety during asynchrony by preventing secret quorums. Intuitively, it does so by bringing the network a step closer to common knowledge - a secondary quorum must know about a preliminary one.
[^4]: There are some caveats here for naive protocols e.g. uncle blocks.
[^5]: One may think of causal independence in a DAG as the correct generalization of simultaneity (beautifully eradicated by Einstein).

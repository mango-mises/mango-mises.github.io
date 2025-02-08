---
layout: post
title: Integrity Matters
tags: [consensus, proof, integrity]
comments: true
mathjax: true
author: IV
---

In my opinion distributed ledgers face three integrity problems:
1. Network integrity
2. Social integrity
3. Computational integrity

In this post I want to dump some thoughts revolving around these problems.

# Network integrity

Network integrity means all observers have the same view of the ledger. It has three components: a consensus protocol, a broad, open network of operators running it, and a broader network of observers ("the people") who monitor the operators.

In my view, a consensus protocol transforms common belief[^1] in some honest threshold of network nodes[^2] into common belief in consistency of views of the ledger by all observers. I think there is formal content to this viewpoint. For instance, in the oral message algorithm from the [Byzantine Generals Problem](https://dl.acm.org/doi/10.1145/357172.357176) paper, it's cool to interpret a chain of messages (a path in each agent's local tree) as a sequence of belief operators. Disregarding introspections, this seems to lead to "practical" common belief.

The operator network must be open to avoid censorship, and broad for robustness and distribution of power.

The observer network must be as broad and cohesive as possible to collectively monitor the operators. The presence of such an "observer hivemind" creates an integrity web which deters operators from misbehaving, as their offenses will quickly become common knowledge.

Network integrity requires a large and cohesive network. The ideal solution is thus a single unified network, without fragmentation or duplicate efforts. Emphatically, it consists not only of "operator" nodes, but also of "relayer" nodes that participate in P2P efforts and contribute to cohesion, and "observer" nodes which may not participate in P2P but nevertheless monitor integrity.

Such a vision quickly runs into a scaling question: Is there tension between security, performance, and network size?

## DAGs

Causality relations are canonically structured as DAGs:
1. Events may be causally independent.[^3] These are path-disconnected nodes.
2. An event may causally depend on multiple independent events. This is a node with several parents.
3. Two events cannot causally depend on each other.

While the context is not yet relevant, in ours, events correspond to blocks. Each view of the world has its own DAG, encoding known information. Since causality is absolute, all such DAGs are consistent in the sense that their superposition is also a DAG, corresponding to the union of information from constituent views.  Note the stark contrast to possible conflicting views of e.g. the longest/heaviest chain. In my opinion, such discrepancies hint that ordering must be founded on full information, i.e the entire ambient DAG as opposed to arbitrarily chosen substructures.

The canonical DAG structure of causality relations precludes inherent bias in favor of sequential monopolies. In our distributed ledger context, there's a very compelling argument _against_ sequential monopolies - they are hugely centralized by definition. In the next section we'll distill this principled intuition into strong arguments in favor of _wide DAGs_ for the sake of social integrity.

The fact _all_ events are naturally structured in a causality DAG inspires a separation between event revelation[^4] and event ordering. Revelation means each node should keep its full view in the form of its local causality DAG. Ordering then converts this DAG into a chain. This yoga is far-reaching and Sybil-independent. In PoS it is beautifully exemplified e.g. by [Narwhal and Tusk](https://arxiv.org/pdf/2105.11827) and [Mysticeti-C](https://arxiv.org/pdf/2310.14821). In PoW it is incarnated in the masterpiece called [DAG-KNIGHT](https://eprint.iacr.org/2022/1494.pdf) (and already in [GHOSTDAG](https://eprint.iacr.org/2018/104.pdf) before it). Below we go into slightly more detail about the PoS and PoW contexts.

## PoS quorums

Monolithic PoS quorum-based consensus protocols periodically gather many operators in a dedicated voting room[^5] until a quorum of stake is reached. Large quorums become bottlenecks due to communication overhead. Small quorums forgo security. These extremes are interpolated via randomly selected committees: small quorums are accumulated until a global stake threshold is reached, granting finality.

Just to make the security point explicit:
* In monolithic PoS quorum-based protocols, a small quorum means consensus is secured by a small portion of stake.
* In PoW, consensus is secured by all of the hashrate[^6].

Such protocols feature a common fan-out → fan-in pattern which obstructs their throughput by tightly coupling transaction dissemination and consensus.
1. (Fan-out.) A block authored by Alice is propagated out throughout the network, where operators individually vote on it.
2. (Fan-in.) Individual votes are jointly consumed by appending (a hash of) a quorum to the next block.
3. Repeat.

Let's see how the revelation/ordering yoga resolves this obstruction. I have in mind two motivating principles: first is the puritanical idea that individual votes are events and should therefore appear explicitly in the event structure instead of being "meta-events"; second is the observation that monolithic voting is overloaded - it signifies both acknowledgement on information and also an opinion about ordering.

In monolithic protocols, individual voting events on a block causally depend only on the block. Naive voting is structured as a fan graph; subtler voting is still structured as a tree. Evidently, a chain structure excludes trees. Thus monolithic protocols _must_ defer voting to "meta-events". To continue, we draw inspiration from longest chain protocols, which are essentially [foot voting](https://en.wikipedia.org/wiki/Foot_voting). A naive first attempt is to generalize from a block-chain to a block-tree, and implicitly interpret each block as a foot vote for its parent. However, the resulting structure diverges due to fanout. But what exactly is happening here? First, these votes are acknowledgements of information and not votes on ordering. Second, the divergence stems from a glaring redundance: Alice should not propagate a block for each acknowledgement - she can batch her acknowledgements in a single block. Thus we arrive naturally at a block-DAG representing the causality relation between all blocks known to a given network view. This DAG accounts for information revelation, and is separate from ordering. Moreover, throughput has substantially increased thanks to removal of ordering from the critical path of block creation and dissemination.

It now remains to order the DAG into a chain of events. To this end there are various protocols. I'll mention [Bullshark](https://arxiv.org/abs/2201.05677) and [Mysticeti-C](https://arxiv.org/pdf/2310.14821).

I'm guessing (!) the DAG paradigm won't improve the _security_ of PoS quorum-based protocols because it hits the same underlying bottleneck, only now it's explicit as the maximal sustainable DAG width. Specifically, I'm guessing the width can't exceed several hundreds of blocks, which is similar to the Tendermint validator set bounds I've heard from the folks at Informal Systems.

## PoW

In my opinion the field of PoW consensus is essentially complete with [DAG KNIGHT](https://eprint.iacr.org/2022/1494.pdf). This amazing paper is the culmination of an admirable intellectual journey going back at least as far as [2015](https://eprint.iacr.org/2013/881.pdf). I'm very lucky to have access to both Yoni and Sutton, and I'll try to write about DAG KNIGHT in the near future.

Interestingly, while Narwhal interprets the causality DAG as a mempool protocol, the word 'mempool' appears neither in DAG KNIGHT nor GHOSTDAG. If I understand correctly, Kaspa clients do include a mempool protocol that propagates transactions. In my opinion mempool protocols are interesting from an incentive viewpoint, as miners don't have an obvious reason for sharing profitable transactions. I also think interpretting the DAG as a mempool protocol may compliment (what I understand of) Yoni's reverse auction idea. I'll write a bit more below.

# Social integrity

Social integrity generally means people are not assholes, but rather adhere to a common belief in some moral code. In our context, I want to "define" social integrity by paraphrasing a folklore quote[^7]:

> Integrity is doing the right thing, even if you can get away with the wrong thing.

Operators collectively control the ledger creation process: they can censor, delay, omit, inject, and order information & transactions. They may act to maximize their own utility at the possible expense of users, or of the truth. I think such "wrongdoing" falls in two classes:

1. (Oracles) Operators control introduction of external information into the ledger. They can e.g. lie about the weather or an exchange rate. The protocol is oblivious to external events and therefore cannot inherently judge validity.
2. (MEV) Operators may manipulate transactions. The protocol can impose limits on ordering[^8], but it is ultimately oblivious to even the simplest manipulations of censorship, injection, and delay.

With neither an oracle for truth nor an omniscient supervisor, we should draw inspiration from economics and specifically from market mechanism.[^9] Prices are not inherent properties of goods. Instead, they are discovered through markets, which facilitate the expression of individual preferences and integrate them into commonly believed "market prices". It makes sense to emulate such a mechanism in-protocol.

The market analogy highlights the drawbacks of sequential monopologies (chains) compared to DAGs. In a nutshell, a market with a single daily-varying stall is inferior to a market with many stalls (wide DAG). This holds even under the unreasonable assumption that prices remain fixed over time, due to the latency of price discovery (or dually, the latency of "correcting" for a crazy seller). A totally unnecessary association is Birkhoff's ergodic theorem asserting equality between space-average and time-average (w.r.t any measure-preserving endo), except space-average is more natural and discovered faster in our context. Going back to reality, where prices change often, sequential monopolies are even worse. I think this jusifies wide DAGs for the oracle problem.

For MEV I want to outline my "noise" approach, which merely obstructs MEV, vs Yoni's beautiful reverse auctions, which actually improve economic efficiency (and life) for both operators and users.
1. (Noise) When ordering causally independent blocks, the protocol should mix the transactions between them. Mixing obstructs MEV since operators cannot even _locally_ control ordering: their proposed blocks are not pasted "as-is" into the ledger. The wider the DAG, the more obstructive the noise becomes. This approach can be adapted to chains at the cost of latency.
2. (Reverse auctions / bilateral market) In a nutshell, if several operators wish to include a given user transaction, they can compete in an auction where each bid signifies the "kickback" offered from the operator to the user. Thus we have a beautiful duality: on one hand operators auction-off ledger space, but on the other hand users auction-off their transactions. The devil is in the precise specification of the auction mechanism. At any rate, this approach too can be adapted to chains at the cost of latency.

A remark on reverse auctions. Ideally, once a transaction is "registered", _all_ operators can enter the auction for its inclusion. I think the canonical way to achieve this is to interpret "registration" as inclusion in the DAG, not something at a naive mempool level. In a layered DAG, the auction can be held for some set amount of rounds. Even fancier things come to mind, like bid escalation (multiple round auctions).

I'm convinced that wide DAGs are crucial for social integrity.

# Computational integrity

Network integrity ensures consistent views of the ledger. Social integrity ensures it is created honestly and with minimal exploitation. But ledgers also involve computation. Be it signature verification or invocation of sophisticated programs, every computation must be executed with integrity. And to be explicit, we want every node in the network to verify computational integrity: operators _and_ observers.

Two keys flows which require verification are syncing a new node from scratch (observer or operator), and monitoring the ledger in the steady-state.

The naive approach to verify computational integrity is to re-execute the computation. This approach is fine when computational demand is low (think of Bitcoin or Kaspa in its current form). However, decentralized computation turns out to be very interesting, with usecases like DeFi and account abstraction. High computational demand reveals a severe problem with the regime of verification by re-execution: increased throughput raises the hardware threshold and thus acts as a centralizing force. Since a _large_ cohesive network is required for network integrity, we are forced to compromise on computational throughput. Ethereum has famously chosen this compromise, limiting computational throughput so new nodes can sync from scratch in a reasonable amount of time.

Validity proofs provide a succinct verification regime, devoid of tension between network integrity and computational integrity. The magic lies in a beautiful assymetry: a prover does extra work (in addition to executing the computation) so that any verifier can _succinctly_ verify computational integrity. I think Sudoku is still the best analogue: the prover works hard to solve a puzzle, and then anyone can _quickly_ verify the solution just by checking rows, columns, and squares. Eli showed me perhaps the most lucid synopsis, taken straight from [Checking Computations in Polylogarithmic Time](https://dl.acm.org/doi/pdf/10.1145/103418.103428):
> In this setup, a single reliable PC can monitor the operation of a herd of supercomputers working with possibly extremely powerful but unreliable software and untested hardware.

To fix terminology, a prover computes proofs of (computational) _statements_, and verifiers are convinced of their correctness upon verifying these proofs. This abstraction makes proofs very convenient for system design, though the devil is in the details. Let's illustrate their power with simple examples.
1. To sync from scratch, a node must verify proofs that collectively assert the computational integrity of the ledger up to some specified point. The concrete statements include not only execution of various smart contracts, but also the consensus itself!
2. To monitor, a node must verify proofs of new blocks.

That's it, right? Wrong! The above applications of proofs facilitate a broad network of _observers_, but it does not resolve centralization of the _operator_ network. Indeed, it seems operator hardware very directly bounds computational throughput. To the rescue comes an elegant observation: prover/verifier assymetry can be used to _defer computation_ to other parties. This is precisely the idea underlying rollups: defer computation away from the settlement layer, and keep only verification.

Proofs are the missing piece for resolving all integrity matters!

In the next post I want to focus on scalable inetgrity webs.

______

[^1]: Common belief in the context of modal logic.
[^2]: It is customary to disregard the distinction between network operators (creators of blocks/votes) and other nodes that can only manipulate the propagation of messages. In practice this distinction is important.
[^3]: One may think of causal independence in a DAG as the correct generalization of simultaneity (beautifully eradicated by Einstein).
[^4]: I like the term "revelation principle" used by Yoni and Sutton.
[^5]: Protocols often use two voting rounds to finalize. The second round ensures safety during asynchrony by preventing secret quorums. Intuitively, it does so by bringing the network a step closer to common knowledge - a secondary quorum must know about a preliminary one.
[^6]: There are some caveats here for naive protocols e.g. uncle blocks.
[^7]: The "original" says "integrity is doing the right thing, even when no one is watching". I have seen it misattributed to C.S Lewis, maybe due to the thematic resonance with his apologist views. At any rate, in our field I hope _everyone_ is watching all the time - the whole is to avoid _hoping_ for integrity.
[^8]: UTXOs have built-in ordering by their very native as "directed envelopes". In Ethereum, each user controls the ordering on its transactions using sequence numbers, known in the Ethereum ecosystem as account nonces. Furthermore, there is literature on fair-ordering protocols that impose causality relations on the ledger.
[^9]: We could and probably should have used exchange rates and not prices, but the passage to cardinals somehow simplifies the subsequent bit about common knowledge in my opinion.

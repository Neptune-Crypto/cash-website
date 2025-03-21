+++
title = "UTXO Types and the Data Availability Problem"

[extra]
author = "Alan Szepieniec"
+++

# UTXO Types and the Data Availability Problem

Scalability and smart contracts are fundamentally at odds with each other. This note explores the problem and articulates the tradeoffs informing Neptune's architecture.

## Introduction

Off-chain payment channel networks such as Lighting promise not only to scale the transaction throughput of cryptocurrencies, but to reinforce privacy as well. The standard approach to make smart contracts private, first championed by [Zexe](https://eprint.iacr.org/2018/962.pdf), is to offchain them as well: the user evaluates the smart contract locally, and generates a non-interactive zero-knowledge proof of correct execution. If the smart contract has a state, then this state is hidden behind a cryptographic commitment and the zero-knowledge proof must demonstrate that it is updated in accordance with the smart contract's logic. In fact, there is no reason not to extend this cryptographic concealment to the smart contract logic itself, also.

![](zexe-style.svg)

Scalability generally refers to one of two notions:
 a) The evolution of workload and hardware requirements for participating nodes as a function of the age or popularity of the protocol. How easy is it to join the network? This refinement might be called *network scalability*.
 b) The evolution of the transaction throughput capacity as a function of the age or popularity of the protocol. How difficult is it to get your transaction confirmed when there are many others competing for confirmation? This refinement might be called *throughput scalability*.

This Zexe-approach does a good job in protecting the users' privacy. It might surprise the reader that it *also* does a good job benefiting scalability -- but only network scalability. Indeed, the non-interactive zero-knowledge proof can be made to satisfy *succinctness*, which is the property that makes verifying a proof an order of magnitude faster than naïvely re-executing the original program. By offchaining the execution of smart contracts, the blockchain protocol becomes more scalable (in terms of network scalability).

However, Zexe-style offchaining is detrimental to throughput scalability. The reason is that *smart contracts are iterative computations*[^1]. Each invocation of a smart contract updates a state. Concealing smart contract states behind cryptographic commitments generates race conditions: updates to this state interfere with each other. The first transaction will make the second invalid because it assumes the old state commitment. The initiator of the second transaction must first obtain the full state that the second transaction outputted. If this information is selectively available then some users have an unfair advantage over others; and if it is lost then the smart contract becomes permanently unusable.

![](iterative-functions.svg)

This, in a nutshell, is the *data availability problem*: the validity of a smart contract may be proved relative to commitments to its state, but its *usability* depends on the dissemination of that state. Pushing state dissemination to a second layer in order to avoid polluting scarce blockchain space is tempting, but this strategy ignores the secondary function of the layer-one consensus mechanism. Consensus does not only validate authenticity; it also replicates data. As such, the consensus mechanism is a robust solution to the data availability problem.

## Transactions in Neptune

Neptune stores coins in UTXOs, which represent singular transfers from one party to another. The alternative to storing coins in UTXOs is to store them in accounts, which are long-lived and generally attached to one user. Taking into account Neptune's architectural design goals, two compelling arguments weigh in favor of the UTXO model:
 1. UTXOs segregate state, can therefore be processed in parallel because there are fewer race conditions, and thus ultimately lead to higher throughput scalability.
 2. UTXOs can be hidden behind cryptographic commitments, and re-randomized and shuffled by a mixnet, thereby reinforcing privacy.

There are two types of UTXOs in Neptune: *lockable* and *lock-free*, depending on whether they have a lock script or not. Lockable UTXOs pass through the mixnet for privacy, whereas lock-free UTXOs are just compactly contained in a Merkle Mountain Range and are reusable by design. Lockable UTXOs represent tradeable assets, whereas lock-free UTXOs are closer to public service announcements -- except that these announcements are not just informative, but *functional* too -- they *do* stuff. Lockable and lock-free UTXOs are functionally equivalent -- the stuff they can do is the same. Moreover, a single transaction can have any number of lockable UTXOs and any number of lock-free UTXOs as inputs.

Both types of UTXOs have the `coins` field, which is a dictionary whose keys are `token_type`s and whose values are `state`s. The `token_type` is found by hashing the type script, which is the logic that determines how the `state` can evolve in the course of a transaction. If the type script of any one of the items in any one of the `coins` dictionaries is not satisfied, then the entire transaction is invalid. 

So far we listed two differences between lockable and lock-free UTXOs: 1) lockable UTXOs have lock scripts, which determine who can spend them; and 2) lockable UTXOs pass through the mutator set for privacy. Difference number 3) relates to how state updates are represented and how this representation affects the consensus mechanism.

Specifically, for lockable UTXOs, states and state updates are private. The transaction validity proofs are zero-knowledge with respect to this information. The `state` field can be the actual state or it can be a Merkle root of which only a few (or even all) leafs are updated. In contrast, the `state` field of lock-free UTXOs are *always* Merkle roots, and *the old leafs, new leafs, and authentication paths are explicitly part of the transaction*.

Transactions in the mempool that interact with the same lock-free UTXO can be tested for compatibility because the state is public. If the transactions are found to be non-exclusionary, they can be joined and cut-through locally -- without having to rely on or wait for the miners to canonize the merger by including it into a block. In particular, this means that the number of transactions per block that interact with a smart contract in a non-trivial way, is limited only by the size of the block.

Note that this solution does not sacrifice the ability of light nodes to succinctly verify the current blockchain state, or their ability to stay up-to-date with minimal effort. The transaction is not valid unless the state update was correct, and invalid transactions can't make it into a block without making its block validity proof invalid. [Full nodes](@/blog/2022-03-08-storage-node-types.md) can opt to forego retaining state for smart contracts they are not interested in. If the network as a whole forgets the state of some smart contract, then that is a tragedy but also supporting evidence that the smart contract in question was not popular enough to bother keeping track of. More importantly, this forgetfulness does not impact the consensus mechanism or the ability of light nodes to synchronize to genesis. Lastly, updates that touch only a small number of memory addresses of a large state, do not needlessly pollute the blockchain because the intact part of the state does not make it into a block. Users wishing to interact only with a part of a large state of a smart contract do not even need to know all of it in order to generate valid transactions.

What *is* compromised is privacy -- or specifically, the *confidentiality* of transactions involving lock-free UTXOs. But that tradeoff is intuitively valid: if you want to rely on the robustness of the consensus mechanism to broadcast, then you should be prepared to sacrifice some privacy. Alternatively, you can participate in a private, off-chain protocol to operate a lockable UTXO with the same type script; but in this case the overlay protocol stipulates how to make the data available to all participants -- or indeed, which subset of participants to make which data available to, or even whether to guarantee availability at all.

![](public-lock-free.svg)

## Summary

 - Neptune uses the UTXO model of transactions and features two UTXO types: lockable and lock-free UTXOs.
 - The data availability problem, which refers to the need for a smart contract's state to be known in order to be useful, is an obstacle for throughput scalability.
 - Lockable UTXOs hide state and pass through the mixnet for privacy. If necessary, an off-chain protocol can be used for data availability.
 - Lock-free UTXOs have public states and leverage the consensus mechanism for data availability in a way that avoids adverse effects on network scalability.

**Acknowledgements**

Thanks to Thorkil Værge, Ferdinand Sauer, and Wei Dai for discussions, feedback, and comments.

[^1]: See also Wei Dai's excellent [exposition of the problem](https://wdai.us/posts/navigating-privacy/), which focuses on privacy rather than scalability. The figures in this note were inspired by (or unceremoniously lifted from) this source.

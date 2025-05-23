+++
title = "Launch Scope and Hard Forks"

[extra]
author = "Alan Szepieniec"
ogdescription = "Scope of features present at launch, and a roadmap for features to be hard-forked in later."
ogimage = "fork.jpg"
ogtype = "article"
+++

|  ![🍴](fork.png)    |
| :-----------------: |
| **Fig.1.**: A fork. |

# Launch Scope and Hard Forks

The [Neptune Whitepaper](../../whitepaper) sketches in high level terms an architecture for next generation money built on top of twenty-first century cryptography. While we have deviated from that plan in select details[^1], by and large our development effort overall has been true to the ideas and ideals set forth therein. However, as we make more progress in developing this project, the full scope of the workload comes into view.

In light of that vantage point, we made the decision to focus on launching a minimum viable protocol as opposed to a feature-complete one. This pivot enables us to respond to market feedback quicker, and acts as a bulwark against premature feature creep.

That said, adding features to a blockchain *ex post* is (or ought to be!) controversial if not downright anathematical. Neptune Cash strives above all to be *sound money*, and that entails having a transparent and fixed monetary policy. Introducing "features" after launch sets the worrying precedent that the protocol can be updated at the whims of elevated authorities, essentially replacing shadowy superbankers with shadowy supercoders. In security terms, a pathway for introducing new features is an attack vector on the money's soundness.

In the interest of transparency, this note outlines which features can be expected to be part of launch and which ones are intended for development afterwards. Furthermore, it captures the opportunity to motivate a *fork policy*, which delineates how new features ought to be rolled out (if at all).

## Hard versus Soft Forks

The terms "soft fork" and "hard fork" come from Bitcoin discourse. The terms are confusing though, inaccurately suggesting that slight changes to the protocol are "soft" and dramatic changes "hard".

A soft fork denotes a change to the protocol that makes the rules for block validity more stringent; and so consequently all blocks produced under the new protocol will be considered valid by the old protocol. When there is a soft fork, the network may split, but as long as the majority of mining power adopts the change, there will still be consensus about the longest chain.

In contrast, a hard fork relaxes the rules for block validity. As a consequence, if there is a hard fork, nodes running the old software will reject blocks produced by the new updated software. The network will necessarily split and there can be no consensus.

On the surface, hard forks seem more powerful in the sense of being capable of delivering more dramatic changes. However, it turns out that soft forks can achieve almost anything that hard forks can. Here is how it is done: the soft fork entails producing parallel blocks, one for each standard block. The nodes that do not upgrade see only the standard block; the nodes that do, see both. The standard block commits to the parallel block by containing its hash. All necessary new validity rules apply to parallel blocks.

For instance, a soft fork could entail doubling Bitcoin's 21 million asymptotic limit. Nodes that do not upgrade would not magically agree to set a different limit. Instead, they would simply not see transactions that are valid under the new limit as those would be contained in parallel blocks. Assuming the majority of mining power adopts the upgrade, old nodes and new nodes would still end up agreeing on the longest chain. Worse still, if the upgrade entails that standard valid blocks contain no transaction data, then refuseniks will be unable to transact with other refuseniks under the old limit. Blocks confirming their transactions will end up losing the block race.

The disqualifying feature of this inflationary soft fork is not technical. Rather, it is sociological: the world in which the majority of mining power agrees to change the supply limit is so far-fetched that one might as well discount it up front.

The difference between soft and hard forks is therefore *not* the quality of upgrades that they can or can not achieve. Instead, the difference is *a)* hard forks necessarily bifurcate the mining power whereas soft forks do not; and *b)* soft forks can ignore and even censor objectors whereas hard forks present them with a choice.

Both features point to soft forks the preferable option (in our eyes). Mining power bifurcations reduce the networks' hashing power and makes them both more vulnerable to attack. The positive way to phrase "objectors are being censored" is "filibusterers are silenced". Proponents are empowered to propose and deploy upgrades without sacrificing the defining feature of money -- its universality as a common medium of exchange.

There are plenty of venues that platform the arguments posed objectors and refuseniks, and we should strive to keep it so. In the end, blockchain money derives its resistance to change – its hardness if you will – not from physics but from the psychological character of its users. The hardest cryptocurrency is the one with the most vital intellectual immune system against bad ideas, and the only way to support and train this immune system is through a lively and open debate.

That said, endless allowance to objectors enables them to filibuster and block progress. Sometimes debates have run their course. Objectors who fail to convince a critical mass on the merit of their arguments should not hold the project hostage and block upgrades. From this perspective, the soft fork is a way to break the deadlock and force their hands: either accept the change, or fork off.

## Hard Forks in Neptune Cash

For succinct blockchains such as Neptune Cash, hard and soft forks are rather more complicated than for non-succinct blockchains. The reason is that every block proves the correctness of its predecessor. Nodes running the "old version" of the client expect blocks with valid proofs, and reject blocks whose proofs are, in their eyes, invalid. Updating the protocol rules by modifying this predecessor-is-valid claim constitutes a hard fork because new proofs are invalid under the old claim. In fact, new proofs are invalid for old verifiers even if the rules become more restrictive such that the new claim implies the old one. Proof systems are not smart enough to understand implications, in contrast to "plain" consensus rules that do not involve proof systems.

Certainly if the proof system is upgraded to a new version, then there is no hope of deploying this upgrade through a soft fork. And unfortunately (and fortunately) we have mathematical improvements in the pipeline for [Triton VM](https://triton-vm.org/) that would make it orders of magnitude more performant, both in terms of speed and memory cost. It would be a shame not to deploy these improvements when they are ready; it would likewise be a shame to delay launch to wait for them; and so we have to bite the bullet and roll out these performance upgrades via hard forks.

What we can do is to make the inevitable hard fork as soft as possible. Even though they were soft forks, the activation of SegWit and then Taproot were instructive examples. Miners signaled their support for the upgrade and when a quorum of mining power was met, the upgrade was locked in, and, after a grace period during which stragglers could update their software, activated.

As for other upgrades to the protocol, there is a sly roundabout backdoor to avoid modifying the predecessor-is-valid claim.

## Soft Forks in Neptune Cash

The trick is to make the predecessor-is-valid claim into a claim about (among other things) undetermined claims[^2].

For instance, the program `PredecessorIsValid` could be a program that *a)* verifies

 - the validity and confirmability of the block's transaction
 - the correct update of the [mutator set](../mutator-sets), which is the data structure that stores UTXOs
 - the correct update of the network control parameters, such as target difficulty
 - the block's predecessor is the genesis block *or* it came with a valid proof for `PredecessorIsValid`;

and *b)*

 - outputs a list of claims that were also verified.

This output list of claims must be part of the block; otherwise there is no way for nodes to verify `PredecessorIsValid`. But besided being contained in the block, there are (initially) no restrictions on these claims.

With this architecture in place, a soft fork can consist of two things. One: a new program, giving rise to a new claim. Two: an extra requirement for upgraded nodes when verifying blocks, namely to verify that the claim in question is present in the list output by `PredecessorIsValid`.

An example helps. Suppose the soft fork involves making all blocks generate an odd number of UTXOs – so blocks that generate an even number of UTXOs are disallowed under the new rules. The oddity of the current block's UTXO count must be verified by a program, say "`GeneratesOddUtxoNumber`". In addition to verifying oddity, `GeneratesOddUtxoNumber` must verify that the block's predecessor is either the last block before which the soft fork was activated, or contains a `GeneratesOddUtxoNumber` claim as one of the claims output by `PredecessorIsValid`. Lastly, upgraded nodes must verify both the `PredecessorIsValid` claim (as they already do) *and* verify that `GeneratesOddUtxoNumber` is part of the output claims.

Major protocol updates can be soft-forked in with this mechanism – *even succinctness.* The bare minimum is for blocks to come with a proof of undetermined claims. `PredecessorIsValid` could be the first item to be added to that list of undetermined claims. Before this soft fork, nodes would need to verify the transaction validity, the mutator set update, and the control parameters updates by scanning history, which is a computationally intense (and non-succinct) task. Strictly speaking, deploying a succinctness soft fork only adds to this task. However, the magic of proof systems makes the computationally intense part of this task superfluous next to verifying the `PredecessorIsValid` proof, as the validity of this proof implies the things that would otherwise be verified with great effort. This can therefore be dropped without affecting the protocol's soundness, and without forking the network.

## Roadmap

Sometimes hard forks are inevitable but often even large protocol updates can be rolled out via soft fork. Whenever a soft fork is possible, that pathway is preferable.

With that context, we can articulate a roadmap of features to be included in the protocol at time of launch, features that would need to be soft-forked in at a later date, and features that would need to be hard-forked in at a later date. This list of features is more like a wishlist than a plan, since changing circumstances might rearrange priorities, and since developing these features requires either volunteers or available financial resources. Furthermore, it is more akin to a snapshot of our current thinking than a commitment to future work – that is to say, priorities can change.

### Scope of Launch

#### Archival Nodes

At launch, Neptune Cash will only support [archival nodes](../storage-node-types), meaning that nodes will download the entire blockchain instead of just the most recent few blocks if they wish to synchronize without making trust assumptions. Moreover, consensus will be defined relative to this historical data. In other words, Neptune Cash will not be succinct.

#### Anonymity

Transactions will be anonymous because the [mutator set](../mutator-sets) is already in place and active. Transactions are proven by their initiators. Transaction proofs can be updated when a block is published that does not confirm it and does not double-spend any of the UTXOs that were confirmed. Transactions can be merged, in which case their two proofs become one. Blocks contain only one transaction.

That said, it is worthwhile pointing out that the standard address format ("generation address") enables linking of transactions to the same address. This linkability is a compromise to reduce the receiver's workload: instead of trial-decrypting all transactions to find transactions destined for him, the user of generation addresses looks for a fixed marker instead. This marker is the same for all transfers to the same generation address.

The simple way to circumvent this issue is to generate a new address for every incoming transaction. A more sophisticated approach involves transmitting encrypted UTXO-information off-band, in which case the marker need not be present in the on-chain transaction. While this feature has not been merged yet, it has been implemented already and lives as a PR; and it will certainly be part of the client at the time of launch.

#### Turing-Completenss

Lock scripts determine who can sign off on UTXOs being spent and under what conditions. Type scripts determine how UTXOs are allowed evolve. To initiate a transaction, the user proves the satisfaction of the lock scripts of the transactions' inputs and of all type scripts involved. These scripts are written in Triton assembler (*tasm* for short), which is Turing-complete and therefore supports arbitrary logic.

#### Transaction Proofs

Every transaction circulates with a single proof of its validity. Furthermore, two such transactions can be merged, in which case a new proof asserts the validity of the merged transaction. Miners apply this procedure on a selection of transactions from their mempools. The net result is that every block contains only one, potentially large, transaction

Note that the term 'transaction' here denotes a bundle of transactions in the standard monetary sense. A Neptune transaction can contain any number of inputs and outputs and can thus involve an arbitrary number of parties.

#### Block Proofs

The soft-fork architecture sketched above, whereby every block contains a proof of a set of undetermined claims, will be in place. That said, at the time of launch, block proofs are unlikely to achieve anything else, contrary to what was sketched above.

#### Two-Step Mining

Two-step mining refers to two tasks that must be done independently as part of mining. First: computing the block proof. Second: guessing the nonce such that the block's hash is smaller than the target difficulty. The block reward is split between the prover and guesser, with the split determined by the prover.

#### Local Control Parameter Updates

Control parameters refer to automatically updated network parameters that regulate some feature, such as the difficulty regulating the block time to track a target interval. Initially the only control parameter is the difficulty, but that may change at a later time.

The important point to note is that the control parameter update is *local*, meaning computed as a function of one block and its successor. This restriction anticipates succinctness, wherein the key selling point is that you don't need more blocks than two down. The alternative to local updates is global updates, whereby the algorithm computes a function of a large window blocks. For example, Bitcoin updates the difficulty every 2016 blocks based on the average block time within that window. Therefore, in order to compute the new difficulty, Bitcoin nodes need (at least) the last 2016 blocks; therefore, Bitcoin is not succinct.

#### Post-Quantum Security

At no point does any code anywhere in the code base rely on cryptographic hardness assumptions that are known to be broken in the presence of quantum computers. It follows that Neptune Cash is plausibly quantum-secure [at genesis](../post-quantum-security-at-genesis).

### Features Requiring a Hard Fork

#### Triton VM Upgrades

At the time of writing, several order-of-magnitude performance improvements are in the pipeline. Their implementation requires a significant engineering effort though, which means in particular that they likely will not be ready by launch. It would be a shame not to upgrade the network when these performance improvements are ready. Unfortunately, this deployment will necessarily induce a hard fork.

#### Security Hazards

Despite all precautions, it is possible that vulnerabilities managed to find their way into the code base. These must be fixed as soon as possible after discovery. Depending on the severity of the vulnerability, this fix may induce a fork. Maintaining consensus is a concern subordinate to having a sound monetary system. In the most extreme case, wherein a vulnerability is discovered that enables undetectable inflation, the network should restart with a protocol fix and a new genesis block.

#### Dynamic Block Intervals and Maximum Sizes

In traditional (single-step) proof-of-work, the tradeoff induced by reducing the target block interval is the greater probability of block races and reorganizations. Specifically, halving the target block time, more than doubles the ratio of time spent disseminating a mined block to the target interval. During dissemination, a competing miner may find their own block and start disseminating it.

In two-step mining such as what Neptune Cash uses, there is an additional complicating factor. The time spent proving a block has essentially the same effect as the time spent disseminating it. While proving, a competing miner may find their own block and start disseminating it. Assuming that all miners have equally performant provers, the same ratio of time spent disseminating blocks to the target interval is still the relevant metric affecting the probability of block races, since it is within this window that hashing power is wasted on chains that are not the longest.

The time spent disseminating blocks is a function of network conditions which can change over time. (Indeed, the [FIBRE](https://bitcoinfibre.org/) network has effectively changed network conditions for Bitcoin miners.) It makes sense to respond to degrading network conditions by increasing the target block interval, with an eye to keeping a constant block race probability. Likewise, if network conditions improve, then the target block interval can be reduced without raising the block race probability.

A similar argument applies to the maximum block size, which is another factor impacting the time spent disseminating blocks. The reason why one might want to increase the maximum block size instead of decreasing the target block interval is because the time spent proving must remain a small fraction of the target block interval in order to ensure fairness[^3]. In particular, when the target block interval has already been reduced to its minimum and network conditions are still good enough, then the maximum block size is another parameter.

Importantly, the determination that network conditions are good enough or not good enough are not made by persons, just like the target difficulty is not updated by a designated person. The protocol stipulates how these parameters are updated as a function of events it observes. In particular, the protocol observes the block orphan rate and it uses this as an indicator of the quality of network conditions. If the block orphan rate is sufficiently low for a sustained period of time, then it may be safe to increase the block frequency or maximum size.

### Features Requiring a Soft Fork

#### Lock-Free UTXOs

To give the public execution rights on a smart contract on Neptune, the creator has to tie it to an anyone-can-spend UTXO. To ensure that it continues to exist even after indefinite future use, the smart contract's type script must require that any transaction that consumes it as an input, must also generate an identical one as an output, up to an updated state.

This construction works, but the down side is that all UTXOs are absorbed into the mutator set upon confirmation. At this point it becomes spendable by only the person(s) in possession of (generally secret) UTXO receiver data. As a result, in order to remain executable by anyone, this UTXO receiver information must be broadcasted somehow. Public announcements can be used to serve this purpose, but this workaround *a)* generates larger transactions and thus higher fees, and *b)* requires type scripts to verify that the necessary data is present in the transaction's public announcement – a potentially expensive task, since the correct execution of the type script is proven. Alternatively, an opt-in overlay protocol could be used for data dissemination without touching the blockchain, but in this case data availability becomes challenging.

A much more elegant solution is for the protocol to natively support another type of UTXOs, *lock-free UTXOs* which, as the name suggests, do not have lock scripts. Since they have no owner, they are are anyone-can-spend by default. And since there is no ownership trace to be deleted, it makes no sense to absorb them into the mutator set – a plain [MMR](../../learn/mmr) will do. In this case there is no receiver data to be broadcasted.

#### Unconfirmed Transaction Chaining

The fact that every UTXO generated by every transaction is absorbed into the mutator set before being emitted at a later date when it is consumed by the next transaction makes it difficult to chain two such transactions. The update to the mutator set must happen in between, which means that a block needs to be confirmed before the second transaction can be. One downside of this architecture is that a smart contract can be executed only once per block.

To support transaction chaining, it must be possible to designate input UTXOs as unconfirmed. Furthermore, it must be possible to generate a single transaction (with a valid proof) from two chained transactions. 

#### Integral Mempools

Unconfirmed transactions live in users' mempools and are broadcasted with proofs. However, while those proofs take up a lot of bandwidth, they need not take up a lot of RAM or disk space. Integral mempools is the reason why.

An integral mempool is a pair `(peaks, proof)` where `peaks` is the peaks list of an [MMR](../../learn/mmr) and where the `proof` establishes that the MMR was only ever updated by adding a valid transaction. With access to such a data structure, a proof of membership establishes transaction validity, and can be used instead of the original transaction proof.

Consequently, upon receiving a transaction, a user running an integral mempool can *a)* verify the proof and prove correct verification; *b)* add the transaction to the integral mempool; and *c)* drop the proof. The net result is that the marginal cost of storing one extra transaction in the mempool is the size of that transaction, without the proof.

#### Succinctness

Succinctness is the property that the fork choice rule can be evaluated between self-professed blockchain tips with at most polylogarithmic supporting data. Neptune Cash will support this feature by requiring blocks to contain a proof of correctness of their predecessors. In this case the amount of supporting data is zero; the fork choice rule can be evaluated instantly. Nodes connecting to the network only need to download the most recent block in order to synchronize with the same robust security that would otherwise require downloading all of history.

### Fork-Independent Features

Except for the features present at launch, the features mentioned above require forks to deploy. It would improper to imply that there no other features. Specifically, there are a bunch of features we would like to see implemented some day, that do not not require forks of any kind to roll out, and that are not high enough priority to work on now. This list is by no means exhaustive, but exists to give an indication of what the fork-requiring features compete with in terms of getting developer attention.

#### Documentation

Documentation describes how the machine works. It is reference material for would-be contributors. Some documentation is already available, but it is by no means complete.

#### Lightning

[Lightning](https://lightning.network/) is the way to scale transaction throughput beyond what rollups are capable of, and do so in a way that does not segregate available capital.

#### Dandelion

[Dandelion](https://arxiv.org/abs/1805.11060) is a transaction broadcast protocol that makes IP data unreliable for deanonymization syndicates aiming to trace the origin of a transaction.

#### Address Types

The current generation address format uses lattice-based cryptography and is post-quantum secure as far as anyone knows. Switching to a different branch of post-quantum cryptography, isogeny-based crypto in particular, will lead to shorter addresses and shorter public announcements (and thus lower fees). The tradeoff is that isogeny-based crypto is newer and less well-studied.

#### User Interfaces

Right now there are two user interfaces, `neptune-cli` and `neptune-dashboard`. The former is a command line interface and the latter a terminal user interface. This situation relegates use of Neptune Cash to the computer-savvy.

#### Applications

A major argument for making Neptune Cash natively support smart contracts is the array of applications that can then be built on top of it. Tokens, bridges, order books, cross-chain atomic swaps, market makers, derivatives, you name it.

## Conclusion

If it were possible to develop all envisioned features at once prior to launch, that course of action would be preferable to the stepwise approach proposed here. However, by being transparent about our intention to fork in features after launch, we mitigate the worst aspect of both soft and hard forks, which is the level of control and authority they bestow unto the developers who deploy them.

Forks *should* be controversial because they tarnish the keystone of sound money, which is that monetary policy should be set in stone and not changed arbitrarily behind closed doors at the whims of privileged elites. By articulating a policy and roadmap we hope to empower a grassroots community to push back against scope creep, whimsical drift, and regulatory capture.

In the end, code is not law – though that is the ideal to be strived for. The fact is that users choose which software to run. The only hope of hard blockchain-based money lies with an informed and critical user base, armed with a capable intellectual immune system against bad ideas. The blockchain protocol itself can at best serve as a focal point for this movement; the body of philosophy that motivates and binds this community is just as important.

> Everyone carries a part of society on his shoulders; no one is relieved of his share of responsibility by others. And no one can find a safe way for himself if society is sweeping towards de­struction. Therefore everyone, in his own interests, must thrust himself vigorously into the intellectual battle. No one can stand aside with unconcern: the interests of everyone hang on the result. Whether he chooses or not, every man is drawn into the great historical struggle, the decisive battle into which our epoch has plunged us.

[^1]: Most notably, when the writepaper was written, [Mutator Sets](../mutator-sets) had not been invented yet. The whitepaper therefore outlines an inferior solution with the same effect.

[^2]: Present authorship notwithstanding, the credit for this ingenious solution goes entirely to Thorkil.

[^3]: Specifically, in order to ensure a mining power distribution that reflects the distribution of cheap energy, which is the consumable resource that makes sybil attacks expensive.
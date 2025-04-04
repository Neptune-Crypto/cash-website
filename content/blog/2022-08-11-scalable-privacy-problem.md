+++
title = "The Problem of Scalable Privacy"

[extra]
author = "Alan Szepieniec"
ogdescription = "Existing privacycoins do not scale. Neptune does. This note explains why."
ogimage = "graph-lightning.png"
ogtype = "article"
+++

# The Problem of Scalable Privacy

Existing privacycoins do not scale. Neptune does. This note explains why.

## Introduction

One of the most important indicators and enablers of success for cryptocurrencies is adoption by active users, for at least two technical reasons. First, a greater user base translates to a greater marketability of the token, in line with a potential definition of "money" as the most marketable good. Second, a greater number of validating nodes reinforces the network protocol's capacity to resist change, reducing the likelihood of a revision to the monetary policy. A certain and predictable monetary policy, in combination with a large user base, enables the transfer of wealth across large spans of time while minimizing risk. In an efficient market, the present-day price reflects the high probability of future wealth preservation.

We refer to this notion, *i.e.*, the capacity to support a large number of participating nodes, as *network scalability*. It is different from *throughput scalability* which optimizes for a certain trade-off between number of transactions per unit of time and the certainty with which they are confirmed. Network scalability is the core motivation behind *stateless blockchains* or *stateless validation*, which is a bit of a misnomer because the objective is not to eliminate the state of validating clients but to minimize it. In particular, the *light client* does not store the entire UTXO set or the entire memory of the virtual machine, which is what would otherwise be needed to validate transactions. Rather, the light client stores only a commitment to the UTXO set or memory that correspond to the tip of the blockchain, and transactions that update them come with supplementary witness data to certify the update, such as Merkle authentication paths. As a tradeoff, transaction-initiating nodes must actively maintain their assets' supplementary witness data.

The discussion about network scalability and light clients is entirely separate from the origin of privacy. Cryptocurrencies are not private by default: the traceability of transactions is what enables an auditor to verify that the books "check out" – that a claimed account balance is correct or that a claimed transfer took place. A branch of cryptography known as [zero-knowledge proof systems](https://zudoku.xyz/) enables an auditor to ascertain the same conclusion with cryptographic certainty without leaking the history of the tokens. If the proof system is *succinct* then the auditor has a trivial amount of work even when the workload of the party generating the proof is substantial.

In theory, it is possible to protect user privacy by integrating zero-knowledge proofs into the design of a cryptocurrency. There are many ways to achieve this integration, and many tradeoffs to consider. The naïve way of integrating zero-knowledge proofs, namely through decoys and nullifier sets, comes at the expense of network scalability. Increasing the workload of network nodes increases the threshold of participation and thus reduces the number of users and the anonymity set. So in practice, *naïvely integrating zero-knowledge proofs into a cryptocurrency design undermines the privacy guarantee that the zero-knowledge proofs were supposed to establish in the first place.*

## The UTXO Set

To understand why privacy technologies in cryptocurrencies might be deleterious to network scalability, it is important to study their effect on the UTXO set. Recall that the UTXO set consists of the set of coins that were output by some transaction but not yet spent by the next. Every transaction consumes some coins from the UTXO set and generates new ones, to be added to the set.

To validate a newly published transaction, a network node needs to verify that the coins that are being spent by it, exist in the UTXO set – because if they don't, then the transaction is either trying to spend new coins into existence, or else trying to double-spend already spent coins. Once the transaction is confirmed, the UTXO set is updated: some coins are removed and new ones are added. In this sense the UTXO set is a snapshot in time of the set of spendable coins. The size of the UTXO set is largely independent of the number of previous transactions.

![](graph-utxo-tx.svg)

**Diagram 1**: The UTXO set in action: to verify that a new transaction (colored blue) spends valid coins, it is not necessary to parse the entire history (colored gray and black) but instead to test the membership of the spent coins to the UTXO set (colored red).

Absent any infrastructure for stateless validation, a full node must store the entire UTXO set in order to perform this membership test. If the UTXO set keeps growing, then so does the cost of participation, and eventually this cost puts downward pressure on the number of active nodes.

Bitcoin's UTXO set seems to be growing in practice. Nevertheless, it is possible for the UTXO set to *shrink* – and the history records several times when this happened. The shrinkage comes about as a result of pruning spent transaction outputs. A prunable UTXO set benefits network scalability, because active nodes do not need to record stale historical data.

With an infrastructure for stateless validation such as [Utreexo](https://dci.mit.edu/utreexo), the cost of participation is trivial for both transaction validators and transaction initiators. The distinction is important:

 - Transaction validators merely validate the transactions initiated by other nodes. The workload of this task is trivial because the inputs' membership to the UTXO set are established by the supplementary witness data (*e.g.*, Merkle authentication path) that the transaction comes with.
 - Transaction initiators need to produce this supplementary witness data when they come into possession of spendable UTXOs, but they must also update this witness data prior to spending them. With an [MMR](../../learn/mmr) the complexity of of these tasks is independent of the prior history, and logarithmic in the number of transactions confirmed in the intervening time.

In summary, the UTXO set is the set of dangling outbound edges in the transaction graph; it is used as a tool for verifying the spendability of coins. All validating nodes need to verify spendability of coins, and so the cost of operating a UTXO set stands in direct relation to the cost of running a validating node. The UTXO set scales independently of the history or popularity of the network. Stateless validation shifts the burden of work from the validators to the initiators of transactions, who must now periodically update their UTXOs' supplementary witness data.

## Privacy Technologies

A cryptocurrency's *lack* of privacy comes from the public visibility of the transaction graph. On the one hand, this transaction graph is needed for auditability, but on the other hand, it is also what enables tracing funds and correlating expenditures. The task of engineering privacy amounts to obfuscating the transaction graph while preserving the auditability through zero-knowledge proofs.

![](graph-amounts.svg)

**Diagram 2**: A transaction graph leaks the movements of money throughout history.

### Confidential Transactions

The first step is to hide transaction amounts behind cryptographic commitments. Every transaction comes with a zero-knowledge proof that the sum of outputs is no greater than the sum of inputs. This construction, known as [*Confidential Transactions*](https://elementsproject.org/features/confidential-transactions/investigation), hides the weights of the edges in the transaction graphs but not their connections. As a result, the analyst who wishes to undermine privacy has no supplementary information with which to cull the search tree and must therefore analyze every possible path.

Confidential Transactions is perfectly compatible with a prunable UTXO set. Instead of adding and removing explicit amounts of cryptocurrency (along with conditions for spending them), commitments to these amounts are added and removed. The core functionality of *prunability* is retained: it is still possible to locate and remove commitments when a transaction that spends them is confirmed.

However, as far as obfuscating the transaction graph is concerned, Confidential Transactions is a half-solution at best. It does not hide the connections of the transaction graph and so it still enables analysts to correlate expenditures if they have side-channel information to help identify or bound transaction amounts. A stronger obfuscation of the transaction graph is needed, in addition to Confidential Transactions.

![](graph-ct.svg)

**Diagram 3**: Confidential Transactions hides the amounts of transaction inputs and outputs. Since the arrows are no longer associated with amounts, the arrows are no longer weighted.

### Decoys

The obfuscation of the transaction graph achieved by Zerocoin, Zcash[^1], and Monero are variations to a common theme that might be called the *decoy approach*. In this construction, every transaction determines a list of possible inputs along with explicit decoys. A separate zero-knowledge proof establishes that *some* inputs are being spent, without leaking which ones.

![](graph-decoys.svg)

**Diagram 4**: With decoys, every transaction determines a list of plausible inputs. Only a subset of plausible inputs are true inputs; the remaining ones are decoys put there to obfuscate the transaction graph.

The decoys must be plausible origins of the money, and so they must be historical UTXOs. In order to prevent double-spending, the decoy approach requires the maintenance of a *nullifier set*. Every UTXO commits to a unique code called the *nullifier*, which remains hidden until the UTXO is spent. When it is spent, the nullifiers produced by the transaction are added to the list of nullifiers. A double-spend can be caught because it generates a duplicate entry in this list. And so in particular, transactions are invalid if they generate a duplicate entry.

The drawback of this approach is that UTXOs can no longer be pruned cheaply. Removing UTXOs makes them unavailable as decoys for future transactions. More importantly, it is impossible for the network to prohibit removing the UTXO when it was not yet spent without undermining privacy. As a result, any removal strategy must either undermine privacy or risk invalidating users' UTXOs even though they were never spent.

Arguably, the unprunability of the UTXO set is not a huge problem in decoy-based privacycoins. Neither the validators of transactions nor the initiators of transactions need to be aware of the entire UTXO set. For the initiators, it suffices to know only an ordered subset or the supplementary witness data authenticating their UTXOs' membership. The validators do not need to know anything about the UTXO set as long as they can verify that the unlocked nullifiers do not already belong to the nullifier set.

The nullifier set is the bigger problem. It cannot be pruned either, because removing elements from this list means allowing double-spends. The burden of work shifts depending on whether the blockchain supports stateless validation, but it is never actually reduced.
 - Suppose the blockchain does not support stateless validation. Then the validator of a transaction must verify that the nullifiers created by it are not already members of the nullifier set. The nullifier set grows with the number of historical transactions, and therefore validating nodes on the network have a **linear** workload as a function of the number of historical transactions.
 - Suppose the blockchain does support stateless validation. Then the initiator of a transaction must prove that the nullifiers created by his transactions are not already members of the nullifier set. The naïve solution, which is to iterate over all nullifiers and prove inequality, has a complexity that is linear in the number of confirmed transactions in the history of the network. A smarter way splits the claim into two components:
   - (1) The nullifier does not belong to the nullifier set at the time when the UTXO was created. The prover can establish the high probability of this claim being true by showing that the nullifier was selected (pseudo-)randomly.
   - (2) The nullifier does not belong to the set of nullifiers added to the nullifier set since the UTXO was created. To establish this claim the initiator of transactions has a workload that is at best **linear** in the number of UTXOs added in the intervening time.[^2]

In either case, the decoy-approach is not conducive to network scalability.

### Mergers

[Mimblewimble](https://scalingbitcoin.org/papers/mimblewimble.txt) proposes an alternative mechanism altogether to obfuscate the transaction graph. It uses Pedersen commitments to hide amounts, but furthermore leverages a peculiar property that these commitments exhibit. When the sum of input amounts equals the sum of output amounts, then the algebraic sum of all contributions happens to have the form of a Schnorr public key. The matching secret key is distributed among the parties to the transaction and by cooperating they can produce a signature authenticating a transaction. 

Glossing over some details[^3], the long story short is that it is possible to merge transactions and hide the fact that before the merger there were two separate transactions. Moreover, if some UTXOs are generated by the one transaction and consumed by the other, these UTXOs can be omitted from the merged transaction altogether. These operations are known as *coinjoin* and *cut-through* respectively. Their effect on the transaction graph is to merge nodes. In the extremal case, every block contains only one transaction but thousands of inputs and outputs.

![](graph-mergers.svg)

**Diagram 5**: Merging transactions results in a transaction graph with fewer edges and fewer nodes.

Merger-based privacycoins are compatible with a prunable UTXO set, and as a result of this compatibility they are conducive to network scalability. It is likely that this promise of greater network scalability fueled the interest in the Grin and Beam projects, both instantiations of the Mimblewimble technology. The apparent loss of interest is likely related to the need to be online and interact with the counterparty in order to send or receive money.

Nevertheless, this interactivity requirement is not an inherent property in merger-based privacycoins. Indeed, the [Neptune whitepaper](../../whitepaper) shows how the same merger can be achieved *non-interactively* with recursive STARKs.

### Miner Mixing

Another strategy for obfuscating the transaction graph charges miners with mixing the UTXO set. UTXOs are still commitments in this constructions, but *re-randomizable* commitments – meaning that anyone can derive a new commitment that is unlinkable but decommits to the same payload. In every block, miners apply this operation to a subset of UTXOs, shuffle them, and include a zero-knowledge proof of correct mixing.

![](graph-mixnet.svg)

**Diagram 6**: Transaction graph when every block mixes the UTXO set.

This strategy is compatible with a prunable UTXO set and therefore conducive to network scalability. It does not generate a workload that scales linearly with the history or time in between receiving and sending money. However, it does increase the workload in some narrow respects:
 - UTXO owners must scan through all outputs of the mixing operation to locate the UTXOs they own.
 - Miners must perform the mixing and prove the correctness of this operation, raising their threshold for participation.

Further more, one downside remains – the decoy approach and the mixing approach have this in common:
 - To avoid wasted scanning work, transaction recipients must be notified by the sender through an off-chain protocol which block their transaction was confirmed in.

In addition to the above drawbacks that relate to workload, there is another drawback that undermines the resulting privacy guarantee to some extent. Specifically, the miner can leak the permutation used in the mix, or sell it to the highest bidder. If this permutation is leaked, the mix generates no privacy. As a result, users seeking privacy who do not trust all miners, must respend received UTXOs to themselves until the UTXOs are confirmed and mixed by a miner they do trust.

Fortunately, Neptune has a solution to this particular problem. But unfortunately, now is not the time to explain how it works. Suffice to say that neither honest nor corrupt miners have any idea about the permutation that underlies the mix they facilitate, and are consequently incapable of leaking this information.

### Lightning

[Lightning](https://lightning.network/) is primarily a technology for throughput scalability, but it benefits privacy as well. The construction involves the establishment of *channels*, which is a UTXO that can either be spent by both parties with mutual agreement or unilaterally to a pre-agreed balance. The parties can agree to update the agreed-upon balance without publishing anything to the blockchain. The balance can be updated any number of times between opening the channel and closing it.

The lightning network consists of nodes and channels between them. Hash time lock contracts enable cascading channel-updates atomically – meaning all at once or none at all. The net effect is being able to send off-chain payments to anyone who is also on the lightning network, provided that there is a path with sufficient capacity.

The only trace that lightning transactions leave on the blockchain is the transactions that open channels, close them, and, unless Confidential Transactions hides amounts, the net difference over the channel's lifetime. The effect on the transaction graph is profound: the part of the transaction graph that is recorded by the blockchain is incapable of bringing into view movements of money on the lightning network. Since lightning transactions do not compete with each other for scarce blockchain space, the lightning part of the transaction graph might be more significant than the on-chain part relative to any meaningful metric.

![](graph-lightning.svg)

**Diagram 7**: Visualization of the transaction graph in the presence of a lightning network. The lightning network is an overlay network, with users (light blue squares) as nodes and channels (light blue circles) as edges. The topology of this overlay network varies from time to time. While a channel exists (full blue line) its balance can be updated (blue circle). By cascading channel updates atomically, participants in the network can make transactions as long as there is a liquid path between them. The only trace of lightning transactions left on the blockchain is the jointly owned UTXOs (colored purple), the transaction that create them, and the transactions that destroy them – all three of which can plausibly be indistinguishable from regular on-chain traffic.

Lightning can work with the account-based model of transactions but is particularly powerful in combination with Confidential Transactions, which works only for UTXOs. In either case, the blockchain must support scripts. Specifically, it must be possible to encode spending conditions that make a transaction invalid a) before a certain time, or b) unless a preimage to a one-way function is provided. The two most popular privacycoins by market cap, Monero and Zcash, do not have this feature.

## Conclusion

Engineering privacy in cryptocurrencies means obfuscating the transaction graph while preserving the auditor's ability to check the books. There are several strategies to effect this obfuscation, with greater and lesser impact on the transaction graph. However, what is rarely considered is how the obfuscation strategy ties into network scalability, which captures the workload of participating nodes and hence the barrier of participation. Naïve obfuscation techniques – even when they employ cutting-edge cryptography to protect privacy – undermine network scalability. The smaller crowds that are capable of participating lead to smaller anonymity sets and thus ultimately undermine the privacy technology's *raison d'ètre*.

Scalable privacy technologies do exist. Neptune uses two of them: Confidential Transactions, which hides amounts behind cryptographic commitments, and mixing a part of the UTXO set in every block. Two more privacy technologies are supported, but not required. First, nodes can merge transactions before publishing them, thereby hiding the original boundaries of separation. Second, Neptune's public scripts are sufficiently expressive to capture the requirements for building a Lightning network.

| technology                | scalable? | Neptune |
|:--------------------------|:----------|:--------|
| Confidential Transactions | ✓         |  ✓      |
| Decoys                    | ✗         |  ✗      |
| Merger                    | ✓         | (✓)     |
| Miner Mixing              | ✓         |  ✓      |
| Lightning                 | ✓         | (✓)     |

**Acknowledgements**: Many thanks to Ferdinand Sauer for proofreading and brainstorming about how to visualize the transaction graph in the presence of a lightning network. Thanks also to Daniel Lubarov for giving useful feedback on a draft.

[^1]: In ZCash the decoy set is the set of all shielded transactions.

[^2]: Some proposals suggest to store the UTXO set in a data structure that admits efficient non-membership proofs such as an [AVL Tree](https://en.wikipedia.org/wiki/AVL_tree). The point remains that this data structure needs to be updated with every added UTXO, and so the workload is still linear.

[^3]: One detail glossed over is the origin of the Fiat-Shamir challenges implicit in the Schnorr signatures.

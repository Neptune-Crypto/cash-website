+++
title = "Why Not Build on Bitcoin?"

[extra]
author = "Alan Szepieniec"
ogdescription = "Why Not Build on Bitcoin?"
ogimage = "neptune-citadel.jpg"
ogtype = "article"
+++

# Why Not Build on Bitcoin?

Why are we launching a new layer-1 instead of building on top of Bitcoin or integrating
our technologies into it? In short, Bitcoin does not currently support trustless
two-way-pegged sidechains, and it is unlikely to adopt our technologies unless we prove their
worth independently. Also, we want to build something that does not inherit Bitcoin's
vulnerabilities.

## Introduction

The question inevitably pops up when we explain what we are building to Bitcoiners: 

 > Why aren't you building on top of Bitcoin or something to improve Bitcoin?

It's a fair question. After all, we are fans of Bitcoin and the architecture of Neptune
is informed by what we perceive are its flaws. Bitcoin has an established track record
and network effect going for it. Surely the distance to "Bitcoin, but better" is shorter 
from "Bitcoin, as it is now" than from yet another altcoin that needs to start from scratch.
Right?

And that's just the practical aspect. There is also Bitcoin's outstanding cultural immune
system, in whose eyes anything that's not Bitcoin is a scam looking to gullible people to
defraud. That heuristic classification might be a valuable work-saving device when it
works accurately in 99.9% of cases, but it doesn't help identify the 0.1% outliers.
More importantly, it tends to turn people who would otherwise be receptive to the values
espoused by Neptune into non-player characters.

## Integrating (Some of) Neptune's Technologies into Bitcoin

Neptune features a bunch of technologies that could in principle improve Bitcoin when
measured in simple straightforward metrics. (Part of the complication is of course that
simple straightforward metrics do not paint the whole picture and there is no widespread
agreement about which set of metrics are adequate.) For instance:

 - *Confidential transactions* hides UTXOs behind cryptographic commitments and proves the
   integrity of transactions (*i.e.*, that the sum of all inputs equals the sum of all
   outputs) in zero knowledge. This construction benefits privacy.
 - *Transaction aggregation* using [Triton VM](../announcing-tvm/) replaces the signatures on a bunch of
   transactions with a single proof authenticating the entire batch, and potentially
   increases the on-chain transaction throughput capacity.
 - *Post-quantum commitments and proofs* – the commitments to UTXOs, the signatures that 
   unlock them, and the proofs that replace the signatures, are all built using
   cryptography that promises to resist attacks on future quantum computers in addition to
   attacks deployed on present or future classical computers.
 - *Recursive block validation* also using [Triton VM](../announcing-tvm/) requires that new blocks prove
   the correctness of the previous block, obviating the need for the costly initial block
   download (IBD) and verification tasks upon first connecting to the network or
   reconnecting after some time spent offline.
 - *Anonymous accumulation* using [Mutator Sets](../mutator-sets/) compresses the UTXO set to a short
   cryptographic digest that can then be used to prove membership of UTXOs to nodes that
   do not store the whole UTXO set, thus reducing storage requirements. Moreover, in order
   obtain this cryptographic digest nodes do not need to compute it from scratch (which
   would require downloading all UTXOs) nor do they need to trust other nodes on the
   network (which would require trust assumptions) because its most recent value is
   present in every block. Lastly, the anonymity of this accumulator benefits privacy by
   making it difficult to link transaction inputs to transaction outputs, a task that
   would be trivial otherwise.

Benefiting privacy, increasing transaction throughput, reducing storage and bandwidth
requirements, are in abstract terms, unequivocal improvements. The challenge is that they
induce controversial tradeoffs in practice. For instance:

 - All these technologies rely on the security of the [Tip5](https://eprint.iacr.org/2023/107) hash function, whereas
   Bitcoin currently only relies on the security of [SHA-256](https://en.wikipedia.org/wiki/SHA-2) and [RipeMD-160](https://en.wikipedia.org/wiki/RIPEMD) (in terms of
   hash functions). Tip5 is not provably secure and is unlikely to be proven secure
   in the future (a feature it shares with SHA-256 and RipeMD-160). Integrating any one of
   Neptune's technologies into Bitcoin therefore implies introducing a new and relatively
   under-tested hardness assumption.
 - Triton VM in particular and multi-round proofs in general are only provable in the
   [random oracle model](https://en.wikipedia.org/wiki/Random_oracle), a simplification of the real world that replaces concrete hash
   functions with an idealized version for the purpose of producing a security proof. 
   So even if Triton VM were to switch from Tip5 to SHA-256, its integration into Bitcoin
   would imply introducing a new cryptographic assumption – not a cryptographic hardness assumption,
   but rather the assumption that the random oracle model is adequate.
 - Even if the security proof in the random oracle model is palatable, Triton VM is 
   instantiated using parameters targeting *conjectural soundness*, a regime of parameters
   where the soundness proof is not known to hold. Nevertheless, no attack is known and
   for all we know a proof for this conjecture may one day be discovered.
 - Even if all the security proofs did go through and in the standard model, that leaves
   the question of implementation errors. Vulnerabilities have been discovered even in
   popular cryptographic libraries years after they were introduced.
 - While verifying proofs instead of individual signatures or blocks reduces the workload
   and resource requirements for one user, this reduction is only possible if another user
   produces the proof to begin with. Without a change to Bitcoin's current incentive
   mechanism, there is no incentive for the proof-producer to work; and so ensuring the
   production of proofs requires changing one of Bitcoin's most delicate and 
   worst-understood components – the incentive mechanism.
 - The use of accumulators requires users to keep the membership proofs of their UTXOs 
   synchronized as the accumulator is updated. Synchronization requires users to be 
   continually online or else spend effort upon rejoining the network to catch up before
   they are in a position to initiate transactions. (This task is relatively benign in 
   terms of complexity but the point is that its complexity is nonzero.)

And these are just the technical tradeoffs. The pathway towards an upgrade to integrate
some or all of these technologies is fraught with difficulties as well.

 - The integration of any one of these technologies would require at least a [soft fork](https://www.investopedia.com/terms/s/soft-fork.asp),
   and thus risks bifurcating the network or the community or both.
 - Any new technology introduced into Bitcoin is there to stay because phasing them out
   would mean eliminating code relative to which historical blocks are valid. Any new
   technology has to be maintained forever. As a result, maintainers understandably want
   100% certainty that the proposed upgrade is the right solution and will not ever need
   to be revised (even as new science is developed).
 - The upgrade would need to enjoy overwhelming support from a large variety of
   developers, miners, influencers, and bitcoin holders. Previous hard forks have shown
   that this level of consensus is exceedingly hard to attain except for the most
   un-controversial changes.

Given that the technologies are controversial, they are very unlikely to ever be 
integrated into Bitcoin. The strategy has a very slim prospect of success and no route to
monetization in the event of success. Under those conditions, who would fund this project?

And to the stereotypical Bitcoiner who answers reflexively: "charity!": even if your goal
is to integrate these technologies into Bitcoin, the best possible argument in favor of
their security and desirability – and therefore the likeliest pathway to success – is the
deployment of a successful cryptocurrency that demonstrates their viability and features.

(That – and also: we await your check.)

## Building a Sidechain on Top of Bitcoin

If integrating some of Neptune's technologies into Bitcoin's layer-1 is infeasible, why
not build them into a layer-2 on top of Bitcoin instead? We could copy the bulk of the
blockchain architecture and even tie the consensus mechanism to Bitcoin's in order to 
avoid a class of problems coming from bootstrapping a proof-of-work system in an
environment full of parties knowledgeable about mining and laden with idle hardware.

In fact, this might be a case where proof-of-stake makes sense: when miners have to earn
block production rights by staking Bitcoin. They are incentivized to stake and produce
blocks because new blocks generate new Neptune coins. Neptune coins, in turn, are valuable
because that's the currency in which the price of the staked bitcoins is declared and 
against which third parties have the option of buying the staked bitcoins at the declared
price. (The system whereby taxees self-declare and tax man has the option to buy the 
goods at the declared price is known as a [Harberger tax](https://en.wikipedia.org/wiki/Harberger_Tax).)
This mechanism has the potential to reel in the tendency of proof-of-stake systems to
devolve into a self-reinforcing oligarchy.

The trouble with this strategy is that *Bitcoin does not support trustless two-way-pegged 
sidechains*.

(That statement is bound to attract some controversy. So we explain below in exact
terms what we want and why existing proposals do not qualify.)

The same difficulty as laid out in the previous section applies – we cannot to rely on an
upgrade to Bitcoin that may or may not happen. Additionally, tying Neptune's consensus
mechanism to Bitcoin's in this way would sacrifice Neptune's claim to crypto-economic security
in the world where quantum computers are built. The following scenario cannot be ruled
out: the quantum attacker dumps stolen bitcoins, the price drops, and miners switch off
their hardware. In this scenario, a side chain Neptune is just as vulnerable as Bitcoin is to 
reorganization attacks.

### Validia Chain

John Light's [excellent report](https://github.com/john-light/validity-rollups/blob/main/validity_rollups_on_bitcoin.md) on the topic calls the sought-after construction a
*validia chain* and even identifies sub-categories that are out of scope here. The user
experience should be some variant of the following:

 1. The user deposits bitcoins into a specially designated Bitcoin address for escrow, by
    making a regular transaction that sends the funds there.
 2. Once the deposit is confirmed, it unlocks an equivalent amount of bitcoins on the side
    chain and assigns them to the user's control.
 3. On the side chain, the bitcoins can be exchanged in a way that benefits from the
    features that the side chain offers (such as privacy and scalability).
 4. When the user wishes to withdraw his bitcoins to the main chain, he burns them while
    announcing a Bitcoin address that.
 5. On the main chain, an equivalent amount of bitcoins are unlocked from escrow and sent
    to the announced Bitcoin address.

The important feature distinguishing a validia chain from a validity rollup (from the
same report) is that a validity rollup uses the main chain for data availability whereas
a validia chain comes with its own data availability solution (and the details about how
this custom solution works maps onto the various subcategories of validia chains).
Validity rollups shrink the footprint of transactions on the main chain, but inherit the
main chain's size limitation. In contrast, the marginal cost of one extra transaction on
the validia chain, as measured by the main chain, is zero.

The common feature shared by both validity rollups and validia chains is that the bitcoins
in escrow are attached to a UTXO that knows or somehow commits to the state of the side
chain. This state can be updated only in accordance with the evolution of the side chain
and only with a valid STARK proof (or transparent SNARK) attesting to this fact. 
Therefore, given the evolution of the side chain, the commitment on the Bitcoin blockchain
evolves integrally without trust assumptions.

In order to support validia chains, Bitcoin requires at least one soft fork upgrade to
enable two distinct features. First, UTXOs need to hold state and bind the ways in which
they can be spent so as to ensure that the state persists and evolves in accordance to
rules set out at the beginning. The technical name for this feature is *covenants*, and
it can be achieved with [`OP_CHECKTEMPLATEVERIFY`](https://bitcoinops.org/en/topics/op_checktemplateverify/). Second, UTXOs bound by a covenant need
to admit updates to the encapsulated state contingent upon a zero-knowledge or 
succinct-verifier (or both) proof. So Bitcoin Script needs a new opcode to verify such
proofs.

### BitVM

An exciting recent development due to Robin Linus called [BitVM](https://bitvm.org/bitvm.pdf) shows that contrary to
popular opinion, Bitcoin is in fact Turing-complete[^1] already. The caveat of this true
statement is that the computation is simulated by an interactive protocol between two
parties. This protocol resembles the dispute resolution protocol in Lightning channels:
the challenger, by exposing the fraud perpetrated by his counterparty, can take all the
bitcoins locked in the channel. In this analogy, BitVM establishes that a state shared
off-chain between two or more parties can be updated in
accordance with pre-defined logic with crypto-economic integrity. Specifically, if one party
updates the state fraudulently, or refuses to supply witness information, the other party
can initiate a dispute resolution process that will end in the fraudster's loss of collateral.

The disqualifying feature of sidechains based on BitVM relative to validity rollups or 
validia chains is the crypto-economic versus cryptographic integrity guarantee.
Specifically, when a zero-knowledge or succinct-verifier proof is required to update the
state (as it is for validity rollups and validia chains) then it is impossible to update
the state to an invalid one unless the updater manages to break the cryptography. By 
contrast, with BitVM it is possible to update the state to an invalid one; the updater
just risks losing their escrow. And since escrow needs to be involved, this updater is in
effect a centralized sidechain operator. Moreover, with BitVM, the sidechain operator can
collude with miners to censor the disputation process. And it might be rational of miners
to collude if their promised share of the spoils is significant.

### Drivechains

[Drivechains](https://www.drivechain.info/) is an older proposal due to Paul Sztorc 
consisting of two BIPs, [BIP-300](https://github.com/bitcoin/bips/blob/master/bip-0300.mediawiki) and [BIP-301](https://github.com/bitcoin/bips/blob/master/bip-0301.mediawiki). The idea is that miners vote on withdrawals.
With enough votes, a withdrawal is approved, and the escrowed bitcoins become liquid
again. The voting process takes some 6 months so that there is enough time to act on
malicious votes through the social consensus layer – for instance by banning blocks that
include them or naming and shaming the perpetrator.

The disqualifying feature of drivechains is similar to that of BitVM: crypto-economic 
security as opposed to cryptographic security. Miners can collude to steal withdrawals
if they are prepared to pay the cost. This effectively limits the volume of bitcoins that
can be withdrawn securely per unit of time.

Furthermore, BIPS 300 and 301 have not been approved yet and it is not clear if they ever
will be.

### Liquid

Liquid involves depositing bitcoins to an escrow address controlled by a federation of
authorities. A withdrawal goes through when a quorum of authorities sign off on it.
Although Liquid denotes a specific side chain – the one spearheaded by Blockstream – it is
nevertheless feasible to build another side chain with a similar setup.

The disqualifying feature is that trusted third parties are not even crypto-economic
security mechanisms – they are attack vectors. Spreading key shares among double the 
number of parties does make an attack more cumbersome but not doubly so, and its effect is
a far cry from that of doubling the security level. Moreover, the authorities have to stay
in sync with $n-1$ other authorities, so in practice the breadth to which key shares can
be spread is limited. Lastly, spreading key shares among trusted third parties obfuscates
the fact that trusted custodians are still needed. History is replete with violations of
that trust, and Bitcoin was supposed to deliver us from them.

## Conclusion

Embedded in the heuristic of Bitcoin's immune system, which classifies all
altcoins as scams, is the implicit allegation that founders and proponents of altcoins
seek to enrich themselves at the expense of Bitcoiners. In this narrative, Bitcoin brought
about the first fair distribution of money and any attempt to emulate or redefine this
distribution is undermining it and thus unfair. Builders of alternative currencies are
particularly guilty because they could have elected instead to build on top of Bitcoin and
reinforce the fair distribution rather than undermine it. This allegation is particularly
easy to direct towards founders of cryptocurrencies devoid of technological merit. (Are
there really 10 000 technological innovations necessitating whole new blockchains, or
are some of them just money grabs?)

In the case of Neptune, this allegation is irreconcilable with an important piece of
countervailing evidence: the size of the premine is 1.98% of the asymptotical limit of the
token supply. This premine is barely enough to fund the development of the project, let
alone enrich its founders. (To anyone contesting this claim: we challenge you to 
fundraise for a new cryptocurrency with double the premine.)

In general we applaud founders who become rich as
a result of their project's success, but we also make an exception specifically for money. Credible neutrality is an
essential quality of good money, and so it has to be a precondition for success. A
lopsided distribution, whether favoring founders or early investors, undermines that
credible neutrality.

The reason why we are not building on top of Bitcoin is purely technical: Bitcoin does not
currently support the features we need. In the future maybe, but we cannot afford to stake
the success of our project on convincing a critical mass of skeptical Bitcoiners.

The narrative of the first and only fair distribution of money is just that – a story. And
it could have been more enthralling if Bitcoin's limitations in practice did not drive 
beneficial technologies elsewhere. The true story is the tragedy of
Bitcoin's first-mover disadvantage: a flurry of science and technology was developed in
the wake of its success with the hope to improve it, but that success has made
it too ossified to benefit from those improvements.

**Acknowledgements:** Special thanks for Louis Guthmann for fruitful discussions and 
suggesting the Bitcoin-staked Harberger tax mechanism.

| ![Neptune citadel](neptune-citadel.jpg) |
|:--:|
| Fig.1: The Neptune citadel will not be built on top of the Bitcoin citadel. Image courtesy if Midjourney. |

[^1]: In fact, BitVM shows something strictly stronger: Bitcoin, in its present state
already, can decide undecidable functions. After all, there is a lookup table that maps
algorithms to the Boolean value that indicates whether or not they halt. This lookup
table has a Boolean circuit, and can thus be encoded into a BitVM contract. The
disqualifier in practice is that we have no means to distinguish Boolean circuits that
compute this function from Boolean circuits that do not.


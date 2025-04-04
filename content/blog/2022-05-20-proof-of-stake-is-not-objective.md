+++
title = "Proof-of-Stake is not Objective"

[extra]
author = "Alan Szepieniec"
+++

# Proof-of-Stake is not Objective

Proof-of-Stake is a proposed alternative consensus mechanism to proof-of-work. Instead of requiring the consumption of energy, proof-of-stake requires miners (usually called *validators*) to put digital assets *at stake* in order to contribute to the block production process. Staking incentivizes them to behave honestly, so as to avoid losing their stake. In theory, with only honest validators, the network will quickly come to consensus about the order of transactions and, therefore, about which transactions are invalid double-spends.

Proof-of-stake has been the subject of much debate. Most criticisms focus on security: does it decrease the cost of attack? Many people also articulate sociological concerns: centralization of power, concentration of wealth, plutocracy, etc.

In this note I articulate a much more basic criticism: *proof-of-stake is inherently subjective*. The correct view of a proof-of-stake blockchain depends on who you ask for it. As a result,
 a) the cost of an attack cannot be calculated in units internal to the blockchain, making security analyses void;
 b) debts cannot be settled between parties that do not already agree on which third parties are trustworthy; and
 c) the final resolution of disputes must come from courts.
 
 In contrast, proof-of-work is an objective consensus mechanism in the sense that any set of related or unrelated parties can come to agreement about which view of the blockchain is correct. As a result, any two economic actors can agree on whether a payment has been made, independently of courts or influential community members. This distinction makes proof-of-work suitable, and proof-of-stake unsuitable, as a consensus mechanism for digital currencies.

## Digital Money and Consensus

### 1. The Problem that Needs Solving

One of the most basic operation that computers can perform is copying information. This operation leaves the original copy intact, and produces an exact replica at essentially no cost. Computers can copy just about anything, as long as it is digital.

However, there are some things that exist purely in the digital realm that *can't* be copied. Things that are both digital *and* scarce. This description applies to Bitcoins for example, as well as to other blockchain-based digital assets. They can be sent, but after sending them the original copy is gone. One might disagree with the reason why the market demands these assets, but the fact that this demand exists means that these digital assets are *useful* as a counterpart to balance exchanges with. Condensed to a single word: *money*.

To achieve digital scarcity, the blockchain protocol replicates a ledger across a network. The ledger can be updated, but only with transactions where:
 1. the owners of the spent funds agree,
 2. the net sum is zero, and
 3. the outputs are positive.
 
Any invalid update will be rejected. As long as there is consensus about the state of the ledger among all participants in the protocol, digital scarcity is guaranteed.

It turns out that achieving consensus is a difficult task. Imperfect network conditions generate distinct views of history. Packets are dropped or delivered out of order. Disagreement is endemic to networks.

### 2. The Fork Choice Rule

Blockchains address this problem in two ways. First, they enforce a complete ordering on all transactions, which generates a tree of alternative views of history. Second, they define *canon* for histories, along with a *fork choice rule* that selects the canonical branch from the tree of histories.

It is easy to derive canonicity from trusted authorities or from a digital voting scheme backed by a citizen identity scheme. However, trusted authorities are [single points of failure](https://nakamotoinstitute.org/trusted-third-parties/) and relying on the government to provide identification services turns the resulting cryptocurrency into a tool of politics rather than one that is independent of it. Moreover, both solutions *assume* agreement about the identities and the trustworthiness of third parties. We want to reduce trust assumptions; ideally we have a solution that derives entirely from mathematics.

A solution for deciding canonicity that derives entirely from mathematics generates the remarkable property that the answer is independent of who computes it. This is the sense in which a consensus mechanism is capable of being objective. There is one important caveat though: one must assume that all parties agree on a singular reference point, such as the genesis block or its hash digest. An objective consensus mechanism is one that enables any party to extrapolate the canonical view of history from this reference point.

*Which* branch of the tree is selected to be canonical is not important; what is important is that all participants can agree on this choice. Moreover, the whole tree need not be represented explicitly on any one computer; instead, it suffices for every node to hold only a handful of branches. In this case the fork choice rule only ever tests two candidate views of history at any one time. Strictly speaking, the phrase *the* canonical view of history is misleading; a view of history can only be more or less canonical relative to another view. Nodes drop whichever branch is less canonical, and propagate the one that is more. Whenever a view of history is extended with a batch of new transactions, the new view is more canonical than the old one.

In order for the network to rapidly converge onto consensus about the canonical view of history, the fork choice rule needs to satisfy two properties. First, it must be well-defined and efficiently evaluatable for any two pairs of views of history. Second, it must be transitive for any triple of views of history. For the mathematically inclined: let $U, V, W$ be any three views of history, and let the infix "$<$" denote the fork choice rule favoring the right hand side over the left. Then:
 - either $U < V$ or $V < U$
 - $U < V \enspace \wedge \enspace V < W \enspace \Rightarrow \enspace U < W$

In order for the ledger to accomodate updates, views of history must be extendable in a way that is compatible with the fork choice rule. Therefore, two more properties are required. First, when evaluated on two views where one is an extension of the other, the fork choice rule must always favor the extended view. Second, extensions of a (formerly) canonical view are more likely to be canonical than extensions of non-canonical views. Symbolically, let $E$ denote an extension and $\Vert$ the operation that applies it. Then:
 - $U < U \Vert E$
 - $U < V \enspace \Rightarrow \enspace \mathrm{Pr}[U \Vert E < V \Vert E] > \frac{1}{2}$

The last property incentivizes honest extenders to focus on extending canonical views as opposed to views that they know are not canonical. As a result of this incentive, distinct views of history that arise simultaneously from honest but contradictory extensions, tend to differ only in their tips, where *recent* events are concerned. The further back an event was logged, the less likely it will be overturned by the reorganization imposed by another, more canonical, view of history that diverges at an earlier point. From this perspective *the* canonical view of history is well defined in terms of the limit [^1] of views of history to which the network converges.

The obvious disqualifier in the previous paragraph is the need for extenders to behave honestly. What about *dishonest* extenders? If the adversary can control the random variable implicit in the probability expression, then he can engineer it to his advantage and launch deep reorganizations with high success probability. Even if he cannot control the random variable but can produce candidate-extensions cheaply, then he can evaluate the fork choice rule locally indefinitely until he finds an early-on point of divergence along with an extension that happens to generate a more canonical branch than any one that circulates.

The missing piece of the puzzle is *not* a mechanism that prevents dishonest extensions. In an environment of imperfect network conditions, it is impossible to delineate dishonest behavior. An attacker can always ignore messages that are not to his liking, or delay their propagation, and claim that the network connection is to blame. Instead, the missing piece of the puzzle is a mechanism that makes deep reorganizations more expensive than shallow ones, and more expensive the deeper they go.

### 3. Cumulative Proof-of-Work

Satoshi Nakamoto's consensus mechanism achieves precisely this. In order to propose a new batch of transactions (called *blocks*), and thereby extend some branch, would-be extenders (called *miners*) must first solve a computational puzzle. This puzzle is expensive to solve but easy to verify, and is thus aptly named *proof-of-work*. Only with the solution to this puzzle is the new batch of transactions (and the history it commits to) a valid contender for canon. The puzzle comes with a knob for adjusting its difficulty, which is automatically turned in order to regularize the expected time before a new solution is found, regardless of the number of participants or the resources they devote to the problem. This knob has a secondary function as an unbiasable indicator of puzzle-solving effort, in some unit that measures difficulty.

The process is open to anyone's participation. The limiting factor is not authority or cryptographic key material or hardware requirements; rather, the limiting factor is the resources one is willing to expend in order to have a chance to find a valid block. The probabilistic and embarrassingly parallel nature of the puzzle rewards the cost-effective miner who maximizes the number of [computations per joule even at the cost of a lower number of computations per second](/blog/in-defense-of-proof-of-work/).

Given the target difficulty parameter (the knob) for every block, it is easy to calculate an unbiasable estimate of the *total amount of work* that a given branch of history represents. The proof-of-work fork choice rule favors the branch where this number is larger.

Miners race against each other to find the next block. The first miner to find it and successfully propagate it, wins. Assuming that miners are not sitting on valid but unpropagated new blocks [^2], when they receive a new block from competing miners they adopt it as the new head of the canonical branch of history because failing to do so puts them at a disadvantage. Building on top of a block that is known to be old is irrational because the miner has to catch up with the rest of the network and find *two* new blocks in order to be successful, a task which is on average twice as hard as switching to the new, longer branch and extending that. In a proof-of-work blockchain, reorganizations tend to be isolated to the tip of the tree of history not because miners are honest but because the cost of generating reorganizations grows with the depth of the reorganization. Case in point: according to this [stack exchange answer](https://bitcoin.stackexchange.com/questions/3343/what-is-the-longest-blockchain-fork-that-has-been-orphaned-to-date), excluding forks following software updates, the longest fork on the Bitcoin blockchain had length 4, or 0.0023% of the block height at the time.

### 4. Proof-of-Stake's Solution

Proof-of-stake is a proposed alternative to proof-of-work in which the correct view of history is not defined in terms of the greatest amount of work spent on solving cryptographic puzzles, but rather defined in terms of the public keys of special nodes called *validators*. Specifically, validators sign new blocks. A participating node verifies the correct view of history by verifying the signatures on the constituent blocks.

The node does have the means to distinguish valid views of history from invalid ones. The point is that a competing block is only a serious contender for the tip of the correct view of history if it has a supporting signature (or many supporting signatures). The validators are unlikely to sign alternative blocks because that signature would prove their malicious behavior and result in the loss of their stake.

The process is open to the public. Anyone can become a validator by putting a certain amount of cryptocurrency in a special escrow account. This escrowed money is the stake that is slashed if the validator misbehaves. Nodes verify that the signatures on new blocks match against the public keys supplied by validators when they put their stakes into escrow.

Formally, in proof-of-stake blockchains the definition of the correct view of history is entirely recursive. New blocks are valid only if they contain the right signatures. The signatures are valid with respect to the public keys of the validators. These public keys are determined by old blocks. The fork choice rule is not defined for competing views of history, as long as both views are self-consistent.

In contrast, the correct view of history in proof-of-work blockchains is defined recursively but not to the exclusion of external inputs. Specifically, the fork choice rule in proof-of-work also relies on randomness whose unbiasability is objectively verifiable.

This external input is the key difference: in proof-of-work, the fork choice rule is defined for any pair of different competing views of history, which is why it is possible to speak of *canon* in the first place. In proof-of-stake, it is only possible to define correctness relative to a prior history.

### 5. Proof-of-Stake is Subvertible ...

Does it matter though? In theory, for two consistent but mutually incompatible views of history to be produced, somewhere someone must have been dishonest. And if they behaved dishonestly, it is possible to find out where, prove it, and slash their stake. And since the validator set at that first point of divergence is not in dispute, it is possible to recover from there.

The problem with this argument is that it does not take time into account. If a validator from ten years ago double-signs mutually conflicting blocks — that is, publishes now a signed contradictory counterpart to the block that was confirmed ten years ago — then the history will need to be re-written from that point onwards. The malicious validator's stake is slashed. Transactions that spend the staking rewards are now invalid, as are transactions downstream from there. Given enough time, the validator's rewards may percolate to a large part of the blockchain economy. A recipient of coins cannot be sure that all dependencies will remain valid in the future. There is no finality because it is not more difficult or costly to reorganize the far past than the near past.

### 6. ... Or Subjective.

The only way to solve this problem is to restrict the depth at which reorganizations are admitted. Conflicting views of history whose first point of divergence is older than a certain threshold age are ignored. Nodes that are presented with another view whose first point of divergence is older, reject it out of hand without testing which is correct. As long as some nodes are live at any given time then continuity is guaranteed: there is only one way the blockchain can evolve if too-deep reorganizations are barred.

This solution makes proof-of-stake a subjective consensus mechanism. The answer to the question "what is the current state of the blockchain?" depends on whom you ask. It is not objectively verifiable. An attacker can produce an alternative view of history that is just as self-consistent as the correct one. The only way a node can know which view is correct is by selecting a set of peers and taking their word for it.

It may be argued that this hypothetical attack is not relevant if the cost of producing this alternative view of history is too large. While that counter-argument might be true, cost is an objective metric and so *whether* it is true depends on external factors, that are not represented on the blockchain. For example, the attacker might lose all of his stake in one view of history but does not care because he can guarantee through legal or social means that the alternative view will be accepted. Any security analysis or calculation of attack cost that focuses on what happens on "the" blockchain and does not take into account the objective world in which it lives, is fundamentally flawed.

Internally to the proof-of-stake cryptocurrency, not only is the cost subjective but so is the reward. Why would an attacker deploy his attack if the end-result is not a payout mechanically determined by his ingenuity but instead a broadcast from the cryptocurrency's official team of developers explaining why they have chosen in favor of the other branch? There may be external payouts — for example, from financial options that expect the price to fall, or from sheer joy of causing mayhem. But the point is that the low likelihood of internal payouts undermines the argument that the market cap of existing proof-of-stake cryptocurrencies constitutes an effective attack bounty.

## Money and Objectivity

Money is in essence the object with which a debt is settled. Settling debt effectively requires consensus among the parties to the exchange, in particular about the currency and amount of money. A dispute will lead to the perpetuation of outstanding claims, and a refusal to do repeat business on equal or similar terms.

Effective debt settlement does not require the entire world to agree on the money. Therefore, a subjective money can be useful in pockets of the world economy where there happens to be consensus. However, in order to bridge the gap between any two pockets of micro-economies, or more generally between any two persons in the world, *global consensus* is required. An objective consensus mechanism achieves that; a subjective one does not.

Proof-of-stake cryptocurrencies cannot provide a new foundation for the world's financial backbone. The world consists of states that do not recognize each other's courts. If a dispute arises about the correct view of history, the only recourse is war.

Foundations that develop and support proof-of-stake blockchains, as well as freelance developers that work for them, and even influencers that do not write code, expose themselves to legal liability for arbitrarily selecting a disfavorable (to the plaintiff) view of history. What happens when a cryptocurrency exchange enables a large withdrawal downstream from a deposit in a proof-of-stake cryptocurrency whose transaction appears in only one branch of two competing views of history? The exchange might select the view that benefits their bottom line, but if the rest of the community — prompted by the PGP signatures and tweets and medium posts of the foundations, developers, and influencers — selects the alternative view, then the exchange is left footing the bill. They have every incentive and fiduciary responsibility to recuperate their losses from the persons responsible for them.

In the end a court will issue a ruling on which view of history is the right one. 

## Conclusion

Proponents of proof-of-stake claim that it serves the same purpose as proof-of-work but without all the energy waste. All too often their support ignores the tradeoffs present in any engineering dilemma. Yes, proof of stake does eliminate the energy expenditure, but this elimination sacrifices the objectivity of the resulting consensus mechanism. And that is okay for situations where only pockets of local consensus suffice, but this context begs the question, what is the point of eliminating the trusted authority? For a global financial backbone, an objective mechanism is necessary. 

The self-referential nature of proof-of-stake makes it inherently subjective: which view of history is correct depends on whom you ask. The question "is proof-of-stake secure?" attempts to reduce the analysis to an objective measure of cost which does not exist. In the short term, which fork is correct depends which fork is popular among influential community members. In the long term, courts will assume the power of deciding which fork is correct, and the pockets of local consensus will coincide with the borders that mark the end of one court's jurisdiction and the beginning of the next.

The energy expended by miners in proof-of-work blockchains is not wasted any more than diesel is wasted fueling cars. The energy in proof-of-work is not wasted; rather, it is exchanged for cryptographically verifiable unbiasable randomness. We do not know how to generate an objective consensus mechanism without this key ingredient.

### Acknowledgements

Thanks to Ren Zhang, Ferdinand Sauer, and Thorkil Værge for feedback and comments.

[^1]: Pursuing a rigorous definition of the *limit of history* leads to an entertaining line of thought. There is a number on the real number line between zero and one, whose binary expansion matches with the representation of the consensus view of history covering not just all of the past, but the present and future as well. More recent blocks are encoded in less significant bits. The sequence of such numbers for truncated histories, where the next element of the sequence encodes one more block than the previous, converges -- so the limit is well defined. This calculus characterizes the whole monetary system as the necessary by-product of calculating an ever more precise approximation. And yet, since this number is not purely axiomatical but depends on the universe, it is more akin to fundamental constants of physics than those of mathematics. One could say that the Bitcoin blockchain is the most precisely measured physical constant in all of science.

[^2]: Assuming instead that miners might purposefully delay the propagation of blocks they find leads to the analysis of *[selfish mining attacks](https://arxiv.org/pdf/1811.09943.pdf)*. A [thorough profit-and-loss calculus](https://arxiv.org/pdf/1805.08281.pdf) finds that this class is only profitable after a difficulty adjustment, and only if the difficulty adjustment algorithm does not take into account the proof-of-work proved by orphaned blocks. (Neptune's difficulty adjustment algorithm *does* take orphaned blocks into account.) This class of attacks do *not* negate the conclusion that reorganizations tend to remain isolated to the tips of history because deeper reorganizations are more expensive.

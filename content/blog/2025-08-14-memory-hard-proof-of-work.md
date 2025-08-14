+++
title = "Memory-Hard Proof-of-Work"

[extra]
author = "Neptune Cash Developers"
ogdescription = "Description and motivation for Neptune Cash's memory-hard proof-of-work puzzle."
ogimage = "neptune-memory-card-game.jpg"
ogtype = "article"
+++

| ![Neptune Cash Memory Cards](neptune-memory-card-game.jpg) |
| :-----------------: |
| **Fig.1.**: Neptune memory-cards puzzle. A close match. |

# Memory-Hard Proof-of-Work in Neptune Cash

**Abstract.** With the launch of its rebooted network, Neptune Cash changed the proof-of-work puzzle. On the legacy network, this puzzle had consisted of a simple partial preimage search: the task is to find a nonce such that the hash of the block is less than a given target. On the rebooted network, the puzzle consists of finding not just a nonce but Merkle authentication paths as well; and in addition to satisfying the hash inequality, the authentication paths must be valid. This article motivates the change and explains in detail how the new proof-of-work mechanism works.

## Memory-Hard Proof-of-Work

Proof-of-work is how systems like Bitcoin and Neptune Cash ensure users have to do expensive work in order to contribute blocks to the blockchain, and consequently how such systems defend against spam and other malicious behavior. However, traditional proof-of-work measures pure hashing power. On this metric, custom hardware (ASICs) outperforms ordinary computers by many orders of magnitude, giving a lopsided advantage to players with easy access to custom hardware providers or with the industrial scale to justify their use.

Memory-hard proof-of-work changes this dynamic by requiring large amounts of memory to even attempt to solve the proof-of-work puzzle. Memory is harder to optimize than pure computation, both in terms of algorithmic cleverness and in terms of hardware. Moreover, typical desktop and laptop users already have access to quantities of RAM that are economically infeasible for custom hardware to provide access to. Consequently, memory-hardness makes super-specialized chips less cost-effective, levels the playing field, and helps deter or at least delay industrial-scale, centralized mining.

## Motivations

The point of proof-of-work is not to make rewriting the blockchain hard – this objective is achieved much more effectively by fixing its value once and for all.

Instead, the point of proof-of-work is to make the blockchain *extendable* while
 - a) treating all peers as equals;
 - b) generating consensus about the correct chain;
 - c) disallowing malicious extensions.

Proof-of-work achieves this set of properties by requiring the verifiable consumption of [objective physical resources](../proof-of-stake-is-not-objective.md) as a precondition to extending the chain. This process is called *mining*. No special keys nor government authority is recognized; users either mine, or decline to. Mining generates consensus because of its verifiabile relation to scarce physical resources: by verifying, disagreeing parties can reconcile their disagreements objectively and without recourse to the qualified opinions of others. And lastly, under the assumption that at least 50% of the mining resources are controlled by honest parties, malicious extensions[^1] are likely to be rejected soon, and certain to be rejected eventually.

The salient feature about partial hash preimage search, the proof-of-work puzzle used in Bitcoin and until recently in Neptune Cash, is that it meets this description in the simplest and most straightforward way. If an argument for the security of this mechanism is convoluted, it must be because security is ill-defined in this context; not because the mechanism itself is difficult to understand or to capture in formal terms. The resource being consumed in the course of mining is energy[^2]. Its consumption can be proven succinctly through the provision of a hash pre-image; one hash evaluation suffices to gauge a metric for how much was consumed. If access to cheap energy is well distributed, then the 50% requirement is satisfied as a result.

In light of the perfect simplicity of this mechanism, any alternative proposal has a high motivational bar to pass before it can be eligible for consideration or adoption. The following motivations may be rather specific to Neptune Cash's case, but they do relate to this generic context.

### Not Decentralized in Practice

We observed a lopsided distribution of mining power. A single entity was responsible for at least 90% of the blocks. Furthermore, whether their mining power was active, depended on the exchange rate. Whenever the price fell below a certain point, this miner would either turn his equipment off or direct them at other chains.

This does not constitute malicious behavior. And to be fair, to the best of our knowledge, this miner never used their disproportionate mining power to deploy selfish mining or reorganization attacks, or anything that does constitute malicious behavior. However, the point is that they were capable of deploying such an attack. Proceeding in this regime means trusting that miner not to cave to temptation in the future.

While aligning the distribution of mining power with the distribution of cheap energy might be a recipe for good decentralization in theory, practice bears out a different conclusion.

### Egalitarianism

The old proof-of-work puzzle aligned the distribution of mining power not purely with that of cheap energy, but to a mixture between that and access to special purpose hardware. This distribution favors industrial-scale mining at the expense of casual individuals. For security in the long run, industrial-scale mining is necessary and even desirable, but in order to get to the long run the project has to pass a bootstrapping phase where adoption is a key intermediate objective. And on this metric, a more egalitarian distribution helps: it is fund and empowering to mine your own blocks, and it transforms the casual miner into a evangelist; whereas the personal investment into an industrial-scale mining operation amounts to little more than a dispassionate profit-and-loss calculation.

To achieve a more egalitarian distribution of mining power, the proof-of-work puzzle must certainly counteract the dynamic that favors specialized hardware and perhaps even at the expense of the dynamic that favors cheap energy. Instead, the proof-of-work puzzle must favor typical hardware that casual miners possess already. Memory-hardness achieves this feature: typical users, operating from their desktop or laptop machines, already have access to more RAM than is available on economically viable special-purpose hardware. While time-space tradeoffs always exist and can always reduce the required memory footprint of a computational task, the point of memory-hard functions is to make any such a tradeoff incur a disproportionate penalty in some other resource, such as time or energy. You *could* compute a memory-hard function on an ASIC with limited memory, but why bother if you can compute it faster on a CPU with access to enough RAM – or even slightly slower, but with dramatically less energy? This dynamic puts CPU mining on par with ASIC mining, or at least levels the playing field.

### Distance to Proof-Production

One of the phenomena observed on the legacy network was the disproportionate effort to optimize guessing versus proving. Even before the single centralized miner arrived on stage, the difficulty was exploding. Meanwhile, transactions were having a hard time being confirmed because not enough nodes on the network were capable of producing the proofs required to confirm it.

Tying proof-of-work to proof-production directly is ill-advised[^3], but the proof-of-work mechanism can be made to resemble proof-production in meaningful ways. In particular, the 40 GB RAM requirement for efficient guessing means that any machine that is capable of guessing efficiently (*i.e.*, without paying the time-space tradeoff penalty) is also capable of producing the most expensive proof type that the network understands. As a result, there is a lower barrier for switching from guessing to proving as soon as network conditions make the second activity more profitable.

### Objections

*The 40GB requirement is an arbitrary number and reflects at best a passing snapshot in time; in the long run, ASICs with access to this amount of memory will be made and mining will become industrial again.* If this change in the proof-of-work puzzle, by offering producers an incentive, accelerates the advent of ASICs with enough memory for proving, then one of its important missions is accomplished. Moreover, even if the timeline for big-memory ASICs is not affected, and the transition to an industrial-scale mining economy inevitable, the argument from egalitarianism remains relevant in the bootstrapping phase.

*By changing the proof-of-work puzzle we have set the precedent that we can arbitrarily pick winners and losers.* It is worth noting that this proof-of-work mechanism change was made in sync with a network reboot that was necessary for [other reasons](../inflation-bug-discovered.md). To the extent that precedents were set, they were set in the context that all bets are off anyway.

*By making the business model of the one centralized miner impossible, there are no more cheap coins to buy.* Price movements are the result of individual choices that ultimately cannot be controlled. Accordingly, projections about prices contribute little to a debate about the technical merits of a proposal.

## Constraints

### Definitely Nakamoto Consensus

A basic requirement for change in general is to not make things worse. In the context of proof-of-work, a change introducing memory-hardness could conceivably degrade the consensus mechanism's security. Therefore, the objective is to find a puzzle that reduces to traditional partial hash-preimage search under the pessimistic assumption that memory is free.

The new mechanism achieves this feature. Concretely, the guesser still has to find a partial preimage such that the block hash is smaller than the target. The only difference is that previously, this partial preimage was just the nonce; whereas now it is the nonce plus some other information, and, crucially, in order for the block to pass the proof-of-work validation, this other information must also satisfy a certain sparse relation.

### (Probably) Provably Memory-Hard in the Random Oracle Model

If it were possible to bypass the memory requirement without paying the tradeoff penalty, then the entire change amounts an exercise in making simple things complicated without changing any fundamental metrics. Moreover, the network would risk of having a lopsided distribution in mining power favoring whoever discovered (and kept secret) the penalty bypass route.

The greatest possible assurance that the tradeoff penalty cannot be avoided is a proof of memory-hardness in the random oracle model. We actually do not have such a proof, just a conjecture that the mechanism really is memory-hard and a strong intuition supporting it. This intuition was built from exploring potential tradeoff strategies and observing dead ends. Finding a proof demands scarce resources that are, at least for the time being, more urgently demanded elsewhere.

### Stateless Guessing

Guessing must be stateless in order to effectively generate consensus.[^4] While true statelessness does is impossible for any process that runs on a computer, the best we can hope for is for a single guess to be nearly instantaneous.[^5] However, a process spanning an instant in time cannot touch a meaningful number of memory cells. Therefore, in order for guessing to be memory-hard there must be a preprocessing phase that initializes memory. The preprocessing phase is allowed to be stateful because the conensus comes from the stateless online guessing phase, which comes after.

### Efficient Verification

Succinctness via recursive block validation is still very much on the roadmap. Consequently, at some future point we will have to implement the proof-of-work validation procedure in Triton-assembler, and prove the integral execution of this validator. Producing this proof should be at most a small task relative to the other tasks involved in recursive block validation. Concretely, this means that:

 - Tip5 must be the underlying hash function;
 - Validating a solution must be near instantaneous.

## Construction

### Data Structures

### Preprocessing

### Online Guessing

## Tradeoffs

### Drop Bottom Layer

### Drop Right Half

[^1]: What even is a malicious extension to a blockchain? A strategy like selfish-mining comes to mind, but the more poignant position is that whatever it means, its opposite (honesty) must prevail eventually under the 50% of resources assumption.

[^2]: Actually, the universe [conserves energy](https://en.wikipedia.org/wiki/Conservation_of_energy) locally and so all energy that enters into the mining process must also exit it. The thing being consumed is not energy *per se* but *usable energy* or [exergy](https://en.wikipedia.org/wiki/Exergy). An equivalent characterization of proof-of-work mining is that it requires the consumption of [negentropy](https://en.wikipedia.org/wiki/Negentropy).

[^3]: Tying proof-of-work to proof-production is, in the abstract, an ideal. To gain an advantage in the proof-of-work game, you must do something that makes proving more feasible. Then the incentive for miners to contribute blocks translates into an incentive to accelerate proof-production, which in turn benefits transaction throughput and a host of other metrics of interest. Unfortunately, tying proof-production to proof-of-work is fraught with practical difficulties:

 - Proof-production is stateful; there is always a percentage capturing the degree of completion. Relative to stateless proof-of-work puzzles, miners are less likely to throw away a proof that is 90% completed when a new block comes in on the off-chance that they can win the block race if they plow ahead and complete their current work. As a result, the network has a greater incentive to delay propagation of proof-of-work solutions, and block races are more common.
 - Proof-production is not a black box; it has an observable state that evolves across many discrete steps. There may be optimizations for singular steps that benefit proof-of-work but break regular proof-production. For example, in FRI-based proof systems, it is possible to record the prover state close to completion, modify one leaf in the Merkle tree, and generate a completely different proof that is still valid with high probability. Crucially, this leads to a strategy for generating $k$ proofs much faster than $k$ times the time to generate one proof. In STIR-based proof systems, it is even possible to avoid the large memory footprint that comes with this stategy.

[^4]: (why guessing must be stateless)

[^5]: In practice, it suffices if the time it takes to make one guess is dwarfed by the time it takes to propagate a winning block.
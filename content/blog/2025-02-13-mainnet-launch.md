+++
title = "Mainnet Launch"

[extra]
author = "Neptune Cash Developers"
ogdescription = "Mainnet Launch Announcement."
ogimage = "rocket.jpg"
ogtype = "article"
+++

|  ![mainnet launch](rocket.jpg)     |
| :-----------------: |
| **Fig.1.**: Perchance's visualization of mainnet launch. |

# Neptune Cash Mainnet Launch: A Milestone in Cryptocurrency Evolution

On February 11, 2025, at approximately 21:00 UTC, Neptune Cash successfully launched its mainnet, marking a significant advancement in blockchain technology. This launch, while delayed by 9 hours from the initially announced time, represents the culmination of years of rigorous development and testing.

## Technical Innovations

Neptune Cash introduces several groundbreaking features:

 - Layer-1 zk-STARKs: Neptune Cash is the first blockchain to integrate zk-STARKs at the core protocol level, enabling efficient verification of complex computations.
 - [Mutator Sets](../mutator-sets): This novel approach achieves anonymity without compromising on succinctness, a crucial advancement for scalable, private transactions.
 - [Post-Quantum Security](../post-quantum-security-at-genesis): All cryptographic elements are designed to withstand potential attacks from quantum computers, future-proofing the network.
 - Smart Contract Functionality: The platform supports arbitrary logic, effectively making Neptune Cash a private smart contract platform.
 - [Two-Stage Mining](../mechanics-of-mining-rewards): a novel market dynamic for incentivizing miners to specialize in distinct economic roles – composers, who produce STARK proofs; and guessers, who contribute proof-of-work.

## [Tokenomics](../tokenomics) and Mining

The total supply of Neptune Coins is capped at 42,000,000, with a pre-mine of less than 1.98%. The protocol implements a halving mechanism every three years, and notably, half of each block reward is time-locked for three years. This structure is designed to create a balanced and sustainable economic model.

Both miner roles, composers and guessers, can earn mining rewards. The composer decides unilaterally which proportion to claim as composer fee and the remainder is automatically left as a guesser fee. Guessers guess on whichever block proposal is most profitable to them. Assuming composers respond rationally to economic forces, the price of a block proposal tends towards the cost of computing it; whereas the cost of guessing depends on the competing guessing power.

## What This Means

The mainnet launch activates several crucial protocol aspects:

 - Mining operations have commenced, enabling participants to contribute to network security and earn rewards.
 - The premine allocations that were set in the genesis block and are being confirmed and reconfirmed with every passing block. The clock on the time-lock period of these allocations has been started.
 - The network is growing. We are observing an increasing number of nodes joining the ecosystem, enhancing its decentralization and resilience.
 - Proof of work difficulty is [rising](https://explorer.neptune.cash/), surpassing initial expectations.

However, we must emphasize that Neptune Cash utilizes novel and untested cryptography. While we have confidence in our technology, we advise users to exercise caution and avoid investing more than they can afford to lose. In the event of significant vulnerabilities being discovered, the protocol may need to restart from scratch.

## Launch Challenges

While the overall launch was successful, we did encounter several unexpected issues:

 - *Last minute security enhancements.* On the day of launch we decided to harden security, in particular in relation to [MMR](../../learn/mmr)s. We remain unaware of exploits targeting the object of our concern, but at the same time, categorically ruling out an attack seems difficult, if at all possible. Given the stakes, we decided to modify the code to harden its security and admit a categorical rule eliminating attack vectors. This process, including the necessary recursive dependency upgrades, resulting in a 9 hour delay relative to what was [announced](https://bitcointalk.org/index.php?topic=5529759).
 - *Default network: "beta".* Despite moving the `#[default]` directive on the enum, the value of the variable `network` remained on "beta" due to an oversight in the [command-line argument processor](https://docs.rs/clap/latest/clap/). Early users had to manually specify the correct network with the CLI argument `--network main`. This issue was resolved in v0.1.1.
 - *Connectivity issues.* In part because of the network mismatch, but also a bunch of other reasons including excessive demand, neither of the two bootstrapper nodes were accepting connections. The early adopters responded like proactive heroes do, by sharing their socket addresses and in some cases even running their own bootstrapper node. There was a point where the developers had isolated themselves from the network! Between the community efforts (thanks!) and the introduction of an IPv6-only bootstrapper nodes, the issues have subsided.
 - *Transaction Processing.* Despite rigorously testing this quintessential use case on all testnets, the first attempt to send funds on mainnet resulted in successful transfer but caused both sender and receiver nodes to crash. Upon reboot, wallets restored correctly with accurate balances, and no funds were lost. This issue is currently being investigated.

These unforeseen challenges highlight the complexity of launching a novel blockchain system. The development team continues to address existing and new issues systematically. We encourage users to report any further issues through our [GitHub repository](https://github.com/Neptune-Crypto/neptune-core/issues) rather than directly messaging developers.

## Future Developments

While the mainnet launch is a crucial milestone, development continues. Future major upgrades will focus on:
 - *Transaction Chaining.* Transaction chaining refers to the ability to build transactions on top of other transactions that have not been confirmed yet. Presently, users can only initiate transactions that spend UTXOs that have been confirmed.
 - *[Lock-Free UTXOs](../data-availability).* Presently, all UTXOs circulate with a lock script, even if that lock script stipulates that anyone can spend. The complication arises from having to maintain and broadcast *sender randomnesses* and *receiver preimages*, two pieces of data that are instrumental in keeping UTXOs private. However, keeping UTXOs private defeats the point of having an anyone-can-spend UTXO, which is how one would build smart contracts that anyone can interact with.
 - *Succinctness.* Succinctness is the feature of a blockchain whose consensus requires negligible resources to verify. To this end, Neptune Cash will leverage [recursive validation](../habemus-verifier) for blocks, which will allow the entire blockchain history to be verified through a single block proof.
 - *Prover upgrades.* Upgrades to the STARK engine, Triton-VM, are planned post-launch, including the integration of novel cryptographic techniques such as [DEEP Commitments](https://eprint.iacr.org/2024/1752).

## Community Engagement

We encourage the community to participate. Developers and researchers are invited to contribute to the project, with bounties available for identifying vulnerabilities or contributing in other ways. The team maintains an open communication channel via [Telegram](https://t.me/neptune_project) for updates and technical support.
+++
title = "Inflation Bug Discovered"

[extra]
author = "Alan Szepieniec and Thorkil Schmidiger"
ogdescription = "Announcement of the inflation bug discovery and future plans."
ogimage = "beaver-printing-money.jpg"
ogtype = "article"
+++

|  ![Beaver printing money](beaver-printing-money.jpg)     |
| :-----------------: |
| **Fig.1.**: Grok having a sense of humor. |

# Inflation Bug Discovered

Dear Neptune Cash Community,

We have discovered an undetectable inflation bug. Details are forthcoming. For now we present the conclusion without justification. We have verified this conclusion with a unit test. The bug was discovered by yours truly at roughly 2025-07-04 16:30 CEST.

We are unaware of any instances in which this bug was exploited. All coin amounts that we are aware of match the emission schedule. However, as the resulting inflation would be *undetectable*, we cannot categorically say the bug has not been exploited. At this point we (neither the developers of the blockchain nor the blockchain itself) cannot offer any certainty about the money supply.

Even if we fix the vulnerability with an immediate hard fork – not that difficult, considering it requires a relatively small change – we would only make exploitation impossible *after* the hard fork at the cost of locking in every exploitation that took place prior to it. In other words, the blockchain would forever have an uncertain money supply, and that fact could never be fixed through a hard fork.

Therefore, we have made the decision to reboot the network from a new genesis block, but with the following difference relative to launching of a brand new cryptocurrency. Baked into the new genesis block is a snapshot of the UTXO set at block 21310. The issuance schedule will be identical but fast-forwarded to its configuration at block 21310. The skipped block subsidies will be allocated to a claims fund. An auditable claim mechanism enables owners of UTXOs in block 21310 to claim their funds on the new chain with proofs of ownership of funds in block 21310, as long as the claims fund is not depleted. If the inflation bug was never exploited, the claims fund is large enough to cover all potential claims. If the inflation bug was exploited, then claimants had better be early, or risk being front-run by the exploiter.

Throughout, we will guarantee that recipients of premine coins will receive their allocations in good order – regardless of whether the inflation bug was exploited or not. 

Please stand by for more details about the bug and about concrete plans for the future.

We want to use this opportunity to thank everyone who is contributing to the Neptune software stack. Everyone who has earned developer bounties will still be rewarded these.

Sincerely,

Alan Szepieniec and Thorkil Schmidiger

P.S.: For the record, the hash of block 21310 is ecfae777da1a6b5ad97d3d793cb64b0cb4262ac5e378d2f4e2a5049e731298e058b2000000000000.

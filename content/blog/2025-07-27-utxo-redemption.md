+++
title = "UTXO Redemption"

[extra]
author = "Alan Szepieniec"
ogdescription = "Details about the UTXO redemption procedure"
ogimage = "utxo-redemption-diagram.jpg"
ogtype = "article"
+++

| ![Schematic diagram of a UTXO redeeming itself from another chain.](utxo-redemption-diagram.jpg) |
| :-----------------: |
| **Fig.1.**: Grok adamantly refusing to imagine an abstract cross-chain UTXO redemption protocol. |

# UTXO Redemption

On July 4 2025, Neptune-core developers discovered a catastrophic [undetectable inflation bug](../inflation-bug-discovered). Following this discovery, the decision was made to reboot the blockchain with a new genesis block. It is not a pure reboot though, because there is a UTXO redemption claims process whereby owners of UTXOs on the old chain can reclaim their balance on the new chain.

## The Claims Fund

Specifically, the block subsidy schedule on the reboot chain is fast-forwarded by 21310 blocks, meaning that the first halving will occur 21310 blocks sooner. The skipped block subsidies are be allocated to a claims fund from which disbursements can be made to parties who can prove ownership of UTXOs at block 21310 on the old chain. In this way, the 42'000'000 coins limit remains.

The claims fund will initially be custodied by the founders. After a while, when the infrastructure exists to support it, they may be moved to a smart contract that can validate the redemption claims and automatically make the disbursements when the claims are found valid. Alternatively, they will be burned. We have not decided yet.

If the vulnerability was exploited before block 21310 (which marks the time the bug was discovered), then the exploiter will be able to produce a UTXO redemption claim that is indistinguishable from authentic. If they choose to exercise this claim, it is possible that the fund runs out before other, legitimate, users have had a chance to exercise their claims. In this case, the fund abides by the "first come, first serve" principle because serving all claimants would undermine the 42'000'000 limit and the only alternative is to pick winners and losers based on different information from which claims are legitimate and which are not.

Another limitation is worth mentioning. Due to the anyonymity mechanism, it is difficult to distinguish premine UTXOs from UTXOs created shortly after genesis. Premine UTXOs are ineligible for redemption because they will be present on the reboot chain as part of the genesis block. However, in order to reliably filter out all claims of premine coins there must be a nonzero false negative rate. Concretely, this means that UTXOs that were created shortly after genesis and never spent since might be unclaimable as well. According to our simulations, the probability that a UTXO is unclaimable drops to below 1% at around UTXO 280, or 70 blocks in.
`
## Exercising Claims

The tools needed to produce and verify a UTXO redemption claim live on branch [`redeem`](https://github.com/Neptune-Crypto/neptune-core/tree/redeem). You must be running this version of the software on a node that is synced to block 21310 (or beyond) on the legacy chain and in possession of unspent UTXOs. Lastly, the node must be running while you run the following commands in another terminal:

 - Set the tip to 21310: `> neptune-cli set-tip ecfae777da1a6b5ad97d3d793cb64b0cb4262ac5e378d2f4e2a5049e731298e058b2000000000000`
 - Produce the redemption claim: `> neptune-cli redeem-utxos`
 - (Optional:) to verify the redemption claim: `> neptune-cli verify-redemption`

By default, the `redeem-utxos` command will create a directory `redemption-claims/` and populate it with a file named something like `48955e77e4e2476eca858bbf9c3b2bc1e683b7426e7abe16538bc417b5e426878561d7b7453f39c8.redeem`. 

Send this file to us. After validating it we will initiate a transaction to send the coins to the address listed in the claim.

Speaking of which, every redemption claim is linked to a receiving address. Without it, the proofs will be invalid. By default, the receiving address is set to the node's 0th generation address, the same one as is produced by `neptune-cli premine-receiving-address`. To customize it, use the `--address` argument on `redeem-utxos`.

## Redemptions

In the interest of transparency, all UTXO redemption claims and their associated disbursements are logged below.

(no redemption claims yet)
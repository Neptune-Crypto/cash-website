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

On July 4 2025, Neptune-core developers discovered a [undetectable inflation bug](../inflation-bug-discovered). Following this discovery, the decision was made to reboot the blockchain with a new genesis block. It is not a pure reboot though, because there is a UTXO redemption claims process whereby owners of UTXOs on the old chain can reclaim their balance on the new chain.

## The Claims Fund

Specifically, the block subsidy schedule on the reboot chain is fast-forwarded by 21310 blocks, meaning that the first halving will occur 21310 blocks sooner. The skipped block subsidies are be allocated to a claims fund from which disbursements can be made to parties who can prove ownership of UTXOs at block 21310 on the old chain. In this way, the 42'000'000 coins limit remains.

The claims fund will initially be custodied by the founders. After a while, when the infrastructure exists to support it, they may be moved to a smart contract that can validate the redemption claims and automatically make the disbursements when the claims are found valid. Alternatively, they will be burned. We have not decided yet.

If the vulnerability was exploited before block 21310 (which marks the time the bug was discovered), then the exploiter will be able to produce a UTXO redemption claim that is indistinguishable from authentic. If they choose to exercise this claim, it is possible that the fund runs out before other, legitimate, users have had a chance to exercise their claims. In this case, the fund abides by the "first come, first serve" principle because serving all claimants would undermine the 42'000'000 limit and the only alternative is to pick winners and losers based on different information from which claims are legitimate and which are not.

That said, it is worth stressing that we have no indication that the vulnerability was ever exploited. We just cannot guarantee that.

Another limitation is worth mentioning. Due to the anyonymity mechanism, it is difficult to distinguish premine UTXOs from UTXOs created shortly after genesis. Premine UTXOs are ineligible for redemption because they will be present on the reboot chain as part of the genesis block. However, in order to reliably filter out all claims of premine coins there must be a nonzero false negative rate. Concretely, this means that UTXOs that were created shortly after genesis and never spent since might be unclaimable as well. According to our simulations, the probability that a UTXO is unclaimable drops to below 1% at around UTXO 280, or 70 blocks in.

## Exercising Claims

The tools needed to produce and verify a UTXO redemption claim live on branch [`redeem`](https://github.com/Neptune-Crypto/neptune-core/tree/redeem). You must be running this version of the software on a node that is synced to block 21310 (or beyond) on the legacy chain and in possession of unspent UTXOs. Lastly, the node must be running while you run the following commands in another terminal:

 - Set the tip to 21310: `> neptune-cli set-tip ecfae777da1a6b5ad97d3d793cb64b0cb4262ac5e378d2f4e2a5049e731298e058b2000000000000`
 - Produce the redemption claim: `> neptune-cli redeem-utxos`
 - (Optional:) to verify the redemption claim: `> neptune-cli verify-redemption`

By default, the `redeem-utxos` command will create a directory `redemption-claims/` and populate it with a file named something like `48955e77e4e2476eca858bbf9c3b2bc1e683b7426e7abe16538bc417b5e426878561d7b7453f39c8.redeem`. 

Send this file to us. After validating it we will initiate a transaction to send the coins to the address listed in the claim.

Speaking of which, every redemption claim is linked to a receiving address. Without it, the proofs will be invalid. By default, the receiving address is set to the node's 0th generation address, the same one as is produced by `neptune-cli premine-receiving-address`. To customize it, use the `--address` argument on `redeem-utxos`.

### Synchronizing

If your node is not already synchronized to the legacy network, you have two options.

 1. Synchronize through the regular peer-to-peer network.
     - Start your node with `--peers 139.162.193.206:19798` (note the port number!) to connect to a node that is guaranteed to serve the right blocks.
     - Block download and synchronization will start immediately.
     - If there is only one node on the network, you might fall into the [sync timeout trap](https://github.com/Neptune-Crypto/neptune-core/issues/634). To get around this obstacle, restart your node every so often.
 2. Download the blocks via bittorrent.
     - Download the [torrent file](neptune-cash-legacy-blocks20250728.torrent) and download the blocks from seeders. (Note that the blocks are located in `home/thv/.local/share/neptune/main/blocks/`.)
     - Untar the archive with `tar -xf` (not the usual `tar -xzvf`).
     - Delete the `databases/` and `blocks/` directory if these are present.
     - Start the node with `--bootstrap-from-directory <path/to/blocks>`.

## Redemptions

In the interest of transparency, all UTXO redemption claims and their associated disbursements are logged below.

|     address (abbrev.)     | amount (NPT)                              | disbursed   |  comment (optional)  | redeem file |
|:-------------------------:|------------------------------------------:|:-----------:|:---------------------|:------------|
| nolgam1tph26le...9vmphz6s |    450.0085550000000000000000000000000000 | genesis     | donation address[^1] | [download](48955e77e4e2476eca858bbf9c3b2bc1e683b7426e7abe16538bc417b5e426878561d7b7453f39c8.redeem) |
| nolgam1autuvle...2xnx3e44 |    101.0900000000000000000000000000000000 | genesis     |                      | [download](a997922a5d826258816b38c7414bc00699f7bc3bf715734f942af0c337cd5eb.redeem) |
| nolgam1drha3sp...vkpqmfqw |      9.8543000000000000000000000000000000 | genesis     |                      | [download](1f46a565fdde633075f0ddfe644a1eb54f18fe7386522093d6f793e4b305fda.redeem) |
| nolgam1tr6ux9n...ay779hmg | 434962.9415060983029299817536415794922500 | genesis     | SafeTrade            | [download](2025-08-05-a.zip) |
| nolgam1tr6ux9n...ay779hmg |      0.0399999999999999983324789473280000 | genesis     | SafeTrade            | [download](2025-08-05-a.zip) |
| nolgam16p0urjd...duwpwt3u | 454949.9000000000000000000000000000000000 | genesis     | SafeTrade            | [download](2025-08-05-b.zip) |
| nolgam1tc9reg0...6spazqzk |    569.5866325000000087264452033904642500 | genesis     | Phoebe               | [download](2025-08-05-c.zip) |
| nolgam1tc9reg0...6spazqzk |   2430.0959325000000087264452033904640000 | genesis     |                      | [download](2025-08-05-c.zip) |
| nolgam1qtqfy6n...3g0ad2ls |   4181.6804965567991310052849205903360000 | genesis     | yeah sure, it was a goal of mine :) just made it | [download](2025-08-05-d.zip) |
| nolgam1qtqfy6n...3g0ad2ls |   4295.1215965567991310052849205903360000 | genesis     |                      | [download](2025-08-05-d.zip) |
| nolgam12e9wmfw...dj8c00y4 |   2586.9907999999998122883048097710080000 | genesis     |                      | [download](2025-08-05-d.zip) |
| nolgam12e9wmfw...dj8c00y4 |   2586.9907999999998122883048097710080000 | genesis     |                      | [download](2025-08-05-d.zip) |
| nolgam15xk75as...3xg9syq0 |     24.4999999900000000000000000000000000 | genesis     |                      | [download](2025-08-05-d.zip) |
| nolgam1jfzfp7g...suh7zhgk |      9.2000000000000000000000000000000000 | genesis     |                      | [download](4ebaf3bf1bd4a713747bd7c9decf7d1d79cf9a9c73ac921df74a0ac7ffbe7a7f4108a5fcb9236e3b.redeem) |
| nolgam1r4rml2s...aspyf53j |  12853.4129593380500636918555308195840000 | future      | Minerlab             | [download](2025-08-06-a.tar.gz) |
| nolgam1r4rml2s...aspyf53j | 176689.2704864789804841314795388928000000 | future      | Minerlab             | [download](2025-08-06-a.tar.gz) |
| nolgam1yqpkezu...kxlz56nj |     64.0000000000000000000000000000000000 | future      |                      | [download](2025-08-07-a.tar.tar) |
| nolgam1yqpkezu...kxlz56nj |  31308.8304072400002434731044968529920000 | future      |                      | [download](2025-08-07-a.tar.tar) |
| nolgam1qtfe60t...gy4uupqu |     56.2702856866377999323154529320960000 | future      | MinerLab             | [download](bb7a5b7b645301e424531ba47d5026c565678401844d577354de4b36747ab8d24df0aa413f328210.redeem) |
| nolgam1qtfe60t...gy4uupqu |   3803.7784541643094782088061377249280000 | future      | MinerLab             | [download](bb7a5b7b645301e424531ba47d5026c565678401844d577354de4b36747ab8d24df0aa413f328210.redeem) |
| nolgam1mav4jjj...tgf7ef2h |   1789.0094119195000385935254654812160000 | future      |                      | [download](d35b1a2f18f8be25a89b2967ff2502de4ac846b31e5c9e2a5cf8715eb3991bc6f10c76a96f3c3661.redeem) |
| nolgam1mav4jjj...tgf7ef2h |  10337.0929999999999999991662394736640000 | future      |                      | [download](d35b1a2f18f8be25a89b2967ff2502de4ac846b31e5c9e2a5cf8715eb3991bc6f10c76a96f3c3661.redeem) |
| nolgam1m8umdar...0vam9ma4 |    535.9000000000000000000000000000000000 | future      | no comment           | [download](982c760a4799b34da53c528c32536d475c2522214f99aabf54c6c4805fb9d127386a9478e1af2f55.redeem) |
| nolgam1hvszeed...lz63upq9 |   2004.1200000000000000000000000000000000 | future      | reclaim steps is a little complex,but community member are warm-hearted help me do that, dev respond quickly, everything goes. all good 😂 | [download](1d658a6372f6f2c067d06dd7024fc919aa21891e558da762d0a831b7095902e.redeem) |
| nolgam1d3m2j7r...328vsju6 |  11674.2801230000000000000000000000000000 | future      |                      | [download](cfc5d0891482e20bf6115c64edd319793f173670ab3e8c01f1bc8cc559fec75.redeem) |


## Misc

 - There is a [community tutorial](../../learn/utxo-redemption-tutorial/) on the process, which might be useful in case you get stuck.

[^1]: https://docs.neptune.cash/contributing/donating.html
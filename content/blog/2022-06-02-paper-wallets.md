+++
title = "Paper Wallets"

[extra]
author = "Alan Szepieniec"
+++

One of the most compelling use cases of cryptocurrencies is the ability to send some to a new address whose corresponding secret key is written on a piece of paper. You should be able to store this paper wallet indefinitely. When you need the money, possibly many years from now, you just download a fresh copy of the client software, boot it, enter the secret key, and *vóilà!* you can access your coins.

This note explains how it is done in Neptune.

# Paper Wallets

A *paper wallet* is a secret key written or printed on a piece of paper for cold storage. The secret key corresponds to a UTXO or many UTXOs containing spendable coins. When a client reads the secret key, it should be capable of spending the UTXOs at the user's direction.

Unfortunately, Neptune presents two complicating factors. First is scalability: instead of storing the entire UTXO set, [light nodes](../storage-node-types) store only a short cryptographic commitment to this set. As a result, UTXOs going into a transaction need a supporting membership proof. The problem is that the paper wallet has no mechanism for keeping this membership proof up-to-date. The second complicating factor is privacy: UTXOs are mixed before they are confirmed in a block. This mixing process is what obfuscates the link between transactions that generate the UTXOs and the transactions that consume them. As a result, neither the sender nor the receiver know ahead of time what the index of the final UTXO is or what it looks like. As a result, there is no simple filter to cull the UTXO set for likely matches.

## Problem

### Privacycoins

A client possesses the coins contained in a given UTXO if
 - the transaction that generates the UTXO has been confirmed in a block; and
 - the client knows the secret information that can unlock the UTXO and satisfy its spending policy.

(For cryptocurrencies supporting light clients, the client additionally needs to know an up-to-date membership proof that demonstrates the UTXO's membership in the current UTXO set. But let's focus on just Privacycoins for now.)

If the cryptocurrency provides privacy for the recipient, then no external party should be able to link this UTXO to the spending address published by the client. The recipient cannot learn of the existence of an incoming transaction by scanning the blockchain and filtering for transactions that spend to the given spending address, because that same strategy would be available to the adversary to break privacy.

Scanning the blockchain and filtering for the given spending address does work to tell the recipient about the new transaction if the currency does not support privacy for the recipient. However, if the unlocking mechanism or the spending policy is non-trivial, the recipient still needs to obtain additional information in order to spend the coins later on.

So either way, the recipient faces the problem of having to obtain additional information. 

The receiver who is continually online can try to unlock every UTXO as they are confirmed. The receiver who was offline and wants to catch up has to try to unlock every UTXO that was confirmed in the meantime -- a potentially expensive task. The typical solution to support light-client receivers is to rely on another round of interaction (that need not take place on-chain) whereby the sender notifies the receiver of the block where the transaction was confirmed. The receiver then only needs to try to unlock the UTXOs confirmed by that block. In short, this solution sacrifices non-interactivity for light clients.

Neptune supports various options on this spectrum. This article focuses on the solution that uses public scripts to notify the receiver of an incoming transaction whose generated UTXOs nevertheless remain hidden from the view of third parties. Alternative solutions involve notifying the receiver off-chain or via a support service, or using public scripts to securely transmit non-identifying secret key material assuming the receiver is continually online.

### Light Clients

Assume that there is a solution for letting the paper wallet user know which UTXOs belong to him. How does the paper wallet keep the membership proof up-to-date? Unfortunately, it can't. There is no other solution but to rely on archival nodes or support nodes to supply this information.

The Neptune protocol cannot guarantee that the required information will be available. This drawback is the result of a conscious design decision: block validity is defined recursively and relative not to the entire blockchain history but only to the block's immediate predecessors. As a result, new clients need only download the latest block and verify it to synchronize. They do not need to download or verify historical data unless they want to become an archival node. And therefore, historical data can be forgotten without affecting the ability of new nodes to synchronize with the network.

Therefore, users of paper wallets need to anticipate the possiblity that the network will not serve them with the information they query. There are several strategies:
 - Hope for the best. If the network is sufficiently charitable, archival nodes will exist and will provide the client who attempts to update paper wallet with the necessary information.
 - Assume the worst. The receiver runs an archival node simultaneously with physically storing paper wallets. When he wants to use the paper wallets, he uses the archival node to supply the missing information.
 - Trust the market. If the network is not willing to supply the requisite information for free, then maybe it is willing to supply it for the right price. Since the data in question might unlock money, there is an economic demand for archive-as-a-service. The machinery is in place for enabling fixed-rate fees without trust assumptions or counterparty risk.
 - Entrust the market. The producer of the paper wallet might want to commission an archival node to ensure the data availability when the receiver tries to spend it.

## Solution

### UTXO and Transaction

To present the solution in a standalone manner it is necessary to review some basic data structures in Neptune.

A lockable UTXO consists of the following fields:
 - `coins`, which is a dictionary mapping `token_type`s to `state`s; and
 - `lock_script`, which determines who can spend it.

A lock-free UTXO does not have `lock_script` field. In fact, lock-free UTXOs have nothing to do with paper wallets and can safely be ignored for the sake of this discussion.

Raw UTXOs do not live on the Neptune blockchain. Instead, commitments to UTXOs do. There is a [mixnet](../mutator-sets) that makes it difficult to link the commitment-to-UTXO that is generated by one transaction to the commitment-to-UTXO that is consumed by the next -- even if the two commitments decommit to the same UTXO.

![](unlinkable-input-output-utxo.svg)

A commitment scheme needs randomness in order to provide [semantic security](https://en.wikipedia.org/wiki/Semantic_security). As a result of incorporating randomness, two commitments to the same *payload* are distinct. In order to open a randomized commitment, the user must supply both the payload and the *decommitment information* -- which might be identical to the randomness used when producing the commitment or deterministically derived from it, depending on the concrete commitment scheme. Therefore, in order to unlock it, *the recipient of a UTXO needs to know both the payload and the decommitment information.*

A transaction consists of the following fields:
 - `inputs`, a list of UTXOs
 - `outputs`, a list of UTXOs
 - `fee`, an amount of native currency earmarked for the miner that confirms the transaction
 - `timestamp`, a number indicating when the transaction was broadcasted
 - `public_scripts`, a list of script objects
 - `proof`, a zk-SNARK of correctness

The transaction is valid if the `proof` field is valid. (The verifier of this proof checks for certain things but they do not matter for the purposes of this discussion.) The block in which the transaction is confirmed can only be valid if all the `public_script`s are valid too. The public scripts can serve as vehicles for public announcement broadcasts -- similar to Bitcoin's `OP_RETURN`. Just set the first instruction to `halt` and set the rest of the program instructions to the broadcast data.

### Paper Wallets via Public Script Notifification

The paper wallet contains a secret seed, from which a public key is derived. A UTXO sent to this public key needs to incorporate it in its `lock_script`. For instance, using [IMLWE](https://eprint.iacr.org/2019/282.pdf) and public parameter `G` we have
```
    secret seed --------> secret key --------> public key
    phrase                a, b                 P = aG + b
```
Then the `lock_script` verifies that the prover knows a witness `(a,b)` such that `P = aG + b`.

The UTXO must be generated by a transaction that contains a `public_script` that broadcasts two pieces of data:
 - the hash of public key `P`
 - a ciphertext, encrypted under `P`

The paper wallet client can scan the blockchain, or query supporting nodes, for transactions whose `public_script`s announce the hash of `P`. By decrypting the corresponding ciphertext, the client learns:
 - the `lock_script`
 - the `coins` being transferred
 - the `randomness` used to produce the UTXO (since it is a *hiding* commitment)

When the UTXO is confirmed and added to the UTXO set it is assigned an index which coincides with the number of UTXOs that were ever added to the UTXO set before it. The paper wallet client who scans the blockchain, or queries supporting nodes, for transactions announcing the hash of `P` will be able to determine this index by reading the block that confirms the transaction.

The paper wallet client must compute the correct decommitment in order to spend these coins. The technical details of this task are rather intricate and best left to another note but it suffices to say here that knowledge of the index, the `lock_script`, `coins`, and `randomness` makes this task easy.

### Tradeoffs

This solution is fully non-interactive but does induce tradeoffs.

 1. There is no privacy from the sender; he can tell if the receiver spent the coins. After all, the sender possesses knows the UTXO and can derive the decommitment information. However, since the sender does not have the secret key `(a,b)` he cannot satisfy the `lock_script` and therefore he cannot spend the UTXO himself. **Edit [2023-10-08]:** This issue has been resolved through the separation of `randomness` into two contributions: `sender_randomness` and `receiver_preimage`. As the sender knows only the *hash* of the `receiver_preimage`, he has no way of knowing when or whether the UTXO he generated was spent by the receiver.

 2. Public scripts pollute the blockchain. However, users pay for the blockchain space their transactions occupy.

 3. If a public key is reused according to this scheme, then external observers will be able to tell that it was the recipient of multiple transactions. Nevertheless, the external observers will not be able to tell when or whether the receiver moves his funds, or how many coins were received in each transaction.

It is possible to mitigate some or all of these drawbacks by sacrificing the strong non-interactivity or by relying on external broadcast channels. However, these alternative transaction types are unsuitable for paper wallets and therefore beyond the scope of this article.

## Conclusion

There are two obstacles to paper wallets.

 1. Light clients need to maintain up-to-date membership proofs. Paper wallet clients must outsource the production of membership proofs somehow.

 2. Privacy: the receiver needs to obtain extra information pertaining to coins he receives in order to spend them.

Any cryptocurrency can support at most two out of the following three features:
  - light clients
  - non-interactive payments
  - privacy for the receiver

Neptune supports any combination of two out of three features. For paper wallets, Neptune sacrifices privacy for address reuse. The public script of a paper wallet transaction contains hash of the receiver's public key, along with a ciphertext transmitting the remaining respending details.

**Acknowledgements.** Many thanks to Ferdinand Sauer and Thorkil Værge for useful feedback on a draft.

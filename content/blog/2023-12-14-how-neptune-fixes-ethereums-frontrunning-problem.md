+++
title = "How Neptune Fixes Ethereum's Frontrunning Problem"

[extra]
author = "Alan Szepieniec"
ogdescription = "How Neptune Fixes Ethereum's Frontrunning Problem"
ogimage = "lightning-based-order-book.png"
ogtype = "article"
+++

# How Neptune Fixes Ethereum's Frontrunning Problem

Frontrunning refers to the practice of beating a victim's trading orders in the race to
reach the marketplace, thereby benefiting from the old price before it changes. The
practice is widespread in decentralized finance, prompting Dan Robinson and Georgios
Konstantopoulos to describe Ethereum as a [Dark Forest](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest),
after the [novel](https://en.wikipedia.org/wiki/The_Dark_Forest) that describes an
environment so hostile that broadcasting one's location spells certain death.

The amount of money lost to frontrunning is difficult to estimate. Cybernews [estimates](https://cybernews.com/crypto/flash-boys-2-0-front-runners-draining-280-million-per-month-from-crypto-transactions/) that
280 million dollars are being lost to frontrunning on Ethereum *every month*. [Flashbots](https://transparency.flashbots.net/)
puts the number at 600 million dollars[^1], but this number describes MEV (miner-extractable value), a generalization
of frontrunning that includes more nefarious exploits.

Naturally, Neptune can fix this.

## Introduction

### How Frontrunning Attacks in Decentralized Finance Work

Suppose that after a long and exhausting search that kept you up late at night, you find a
cycle of digital assets along with matching liquidity pools and exchange ratios such that
if you run through all the trades – trade A for B, B for C, and so on, and eventually
something for A – you end up with a profit. This is an arbitrage opportunity; an option
for risk-free profit. It's a competitive environment and you only managed to find this
cycle by employing the right combination of cleverness and computational power. Well done!
But you have to be quick, because before long someone else will stumble across this cycle.

So the last thing you do before you go to bed is broadcast the transaction. And then you
sleep. It was a long day, after all.

Unfortunately, your transaction will not be mined and you will not be able to rake in the
profits you were dreaming of.

The issue is that before your transaction can be executed you need to broadcast it in
order for miners (or stakers) to include it in a block for confirmation. But there is a
window of opportunity between broadcast and confirmation that the attacker can exploit.
Since the transaction was broadcast, the attacker now knows about it. He can 
copy your sequence of trades and change only the end-beneficiary so as to benefit himself
instead of you. And he can bump the fee (or gas cost) so as to guarantee that rational
miners (stakers) prefer *his* transaction instead of yours.

Of course, rational miners are in on the game too and will replace the frontrunner's
transaction with their own.

The problem is not limited to arbitrage trades; arbitrage just makes the benefit obvious. 
Whenever you initiate a transaction, frontrunners have the opportunity to insert a
transaction with preferential treatment.

In terms of trades, they can benefit from the
prices as they are before your trade affects them, or even engineer a price that benefits
them at your expense. The price you end up trading at is rarely as beneficial as
the price you observed when you initiated the trade.

### Decentralized Trading with Confidential Transactions

Automatic trades on transparent blockchains suffer from drawbacks that make them
inherently vulnerable to frontrunning attacks.

 1. Trades and the state of the market are transparent.
 2. Trades are broadcasted.
 3. There is a delay between broadcast and confirmation.

The delay comes from the fact that stakers cannot afford to propose blocks that contain
conflicting transactions, so they must spend time filtering out conflicting transactions
before they propose blocks. Filtering is a time-consuming process.

The need for trades to be broadcasted comes from the architecture of blockchains. Users do
not know beforehand who will mine or propose the block that confirms their transaction. 
The best strategy to ensure that it is confirmed is to ensure that everyone capable of 
confirming it knows that it exists.

The transparency of trades and the state of the market is emphatically *not* an inherent
drawback of cryptocurrencies. Indeed, transaction confidentiality is one of
Neptune's major selling points.

Can Neptune's confidential transactions deter frontrunning attacks in decentralized
markets? As one of the key resources available to the attacker is missing, it would seem
that the answer could be "yes". A problem arises though when marrying confidential
transactions with price discovery.

### Automation

In the case of automatic market makers such as [UniSwap](https://uniswap.org/),
there is another compounding factor:

 4. Trades are executed *automatically*, as a by-product of mining (or staking).

Automation induces dynamics that go a long way towards explaining why automatic market
makers are so popular *despite* the prevalence of MEV.

 - The only requirement for the market to work is liquidity. Investors are rewarded for
   providing liquidity with a passive income at very low risk.
 - The price is discovered automatically, as a by-product of traders chasing arbitrage 
   opportunities.
 - Trades are not matched with counterparts. In particular, the discovery of 
   counterparties is not a precondition for the trade to be executed.

These problems exacerbate MEV. The presence of liquidity and the automatic price discovery
benefits users but also frontrunners. Users do not need to find counterparties but neither
do frontrunners.

These features require amending to the motivating question: can Neptune's 
transactions deter frontrunning attacks on *automatic* decentralized marketplaces? The 
answer here seems to be "no" because the public nature of the state of the market is
a necessary quality for its automatic evolution. In other words, *MEV is the cost of
automation.*

Nevertheless, the features unlocked by automation are enticing and the question remains 
whether confidentiality (or other features) can deter frontrunning attacks
while achieving a subset of them.

## An Architecture for Confidential Marketplaces

It should not be left unsaid that any smart contract deployable on Ethereum can be 
simulated on Neptune simply by exposing the state of UTXOs, which is otherwise concealed.
The question is whether confidentiality leads to strictly better features in terms of
MEV-resistance.

### Two-Way Peg

Since the bulk of the capital for decentralized finance lives on Ethereum either as Eth or
as ERC-20 tokens, it is imperative to build a two-way bridge pegging the price. Otherwise,
we end up building an equally potent or perhaps more potent marketplace but without
addressing the problem that exists on Ethereum.

A two-way bridge is relatively easy to achieve, in part because both blockchains support 
arbitrary logic.

From the point of view of the Ethereum blockchain, there is a smart contract whose
interface supplies two end-points:

 1. `deposit`: allows the user to deposit ERC-20 tokens and bind them to a [Tip5](https://eprint.iacr.org/2023/107) digest,
    such that whoever knows the preimage of this digest can conjure a matching amount of
    tokens on Neptune.
 2. `withdraw`: allows the user to withdraw ERC-20 tokens if he supples (a) proof of a
    prior deactivation transaction on Neptune, and (b) a signature valid under the public
    key stipulated by that withdrawal transaction.

Meanwhile, on Neptune there is a [lock-free UTXO](../data-availability) denoted `bridge`,
which represents the other bridge head. Its type script stipulates that it can be
spent in one of two ways:

 1. `activate` : (`bridge`; `deposit-proof`, `ownership-proof`) --> (`bridge`, `token`)
    Here `bridge` and `token` denote UTXOs, and `deposit-proof` and `ownership-proof` 
    denote [TritonVM](https://triton-vm.org/) proofs. The first proof establishes that 

     - a deposit has been made to the Ethereum bridge head;
     - this deposit has not been redeemed on Neptune yet.

    The second proof establishes that

     - the deposit transaction defines a digest and the transaction initiator knows the 
       matching preimage.

    The `token` UTXO is a lockable UTXO. Its typescript validates simple amounts but also
    uniquely identifies the bridge head as well as the ERC-20 token type.
 2. `deactivate` : (`bridge`, `token`; `pubkey`) --> (`bridge`)
    The UTXO `token` is destroyed. This destruction enables the withdrawal of an equal 
    amount of tokens from the Ethereum bridge head.

Once ERC-20 tokens have crossed the bridge onto Neptune, they can be exchanged there. At
that point they are indistinguishable from any other asset on Neptune, including from
Neptune tokens, and therefore benefit from Neptune's strong transaction confidentiality.

### Atomic Swaps on Neptune

Exchanging ERC-20 tokens on Neptune like any other Neptune asset is the first step. The
next step is to ensure that trades can take place without counterparty risk.

This is easier done than said: Alice and Bob simply *both* sign off on a transaction that
puts the agreed swap into effect. If all goes well, they broadcast the transaction and it
is confirmed shortly after.

If for whatever reason Alice fails to provide Bob with her timely signature even though
Bob provided his, Bob should cancel the swap by re-spending his tokens to himself. Either
(a) Bob's cancellation is confirmed and neither party incurs a loss (except for 
transaction fees), or (b) the swap is confirmed after all. Cancellation is necessary for
Bob to ensure that at some later point when he wants to spend his tokens somewhere else,
no interference from Alice can prevent him.

Confidential atomic swaps cannot be detected and thus cannot be frontrun either. However,
the mere capacity to do atomic swaps privately is not enough: the user also needs to find
a counterparty and a reasonable price.

### Decentralized Counterparty Discovery

Counterparty discovery is the process of finding another party to trade with. In abstract
terms, what is needed to achieve this is a forum where market players can announce their
bids and connect with other market players. A *digital* forum serves this purpose, and
indeed the first realizations might be literal deployments of [phpBB](https://www.phpbb.com/)
or [freedit](https://github.com/freedit-org/freedit). Down the line forum administrators
might condense their application to a feed of announced bids that integrates as a plugin
with the [Neptune core](https://github.com/Neptune-Crypto/neptune-core) node.

The need for a forum highlights an apparent contradiction between the primary feature of 
any serious cryptocurrency – decentralization – and the need for centralization to 
disseminate information efficiently. If everyone runs their own forum they either repeat
each other or they disseminate information from disparate data sets. All other things
being equal, traders will find counterparties faster on the forum with more traffic. Fora
benefit from network effects and therefore have a tendency to centralize.

Two remarks are in order.

 1. The word "decentralization" sometimes denotes the distribution of authority rather 
    than that of information or computing power. While fora do centralize information,
    they are not hierarchical because anyone can start their own.
 2. Fora are non-exclusionary. Traders can subscribe to multiple, although every bid
    can only be matched to at most one counterparty, and if traders announce their bids
    on multiple fora they might disappoint a larger number of would-be counterparties.

Bid-feeds do not necessarily leak volume information; only price. Two parties who connect
via a bid-feed based on proposing compatible prices can agree on the volume through a
private channel.

### On-Chain Bidding

Price discovery refers to the computational process of finding the market-clearing price,
which is where which supply equals demand. The inputs to this process cannot be *declared*
bids but have to be *committed* bids, because otherwise market players can manipulate the
price for free. The output is the price that maximizes the trade volume.

An order book starts with a pair of lists of bids ordered by price, one for buy orders and
one for sell orders. By iterating over all the prices it is trivial to find the price (or
technically: the *interval* of prices) where supply equals demand. Whatever mechanism
traders used to commit to their bids is activated for the sell orders above the price and
for the buy orders under it. This activation is part and parcel of the order book; the
term denotes the marketplace rather than the data structure or the algorithm.

On Neptune, committed bids can be simulated through specially prepared transactions. 
Suppose that the trader who possesses $X$ number of A tokens and wants to trade them for
$Y$ number of B tokens (or more). He generates, signs, and broadcasts a *bid* transaction
which consumes the UTXO consisting of $X$ A tokens and creates a new UTXO also consisting
of $X$ A tokens but with an additional coin ("`bond`") whose typescript makes it spendable
only:

 - by the trader himself;
 - by any transaction that allocates to the trader's public key $Y$ B tokens.

In either case, the `bond` coin disappears after being spent.

As long as no order book operator has spent this UTXO, the trader can cancel the bid 
by using the first spending policy. Whenever a collection of mutually satisfying bids 
exist, anyone can play the part of the order book operator by generating market-clearing
transaction that matches the bids.

Order books are less prone to frontrunning attacks compared to market makers because the
only exchange that can go through is the one the trader consented to. However, there is 
still an unexpected factor, which is when and whether the bid will be matched. Knowledge
of the announced bid may cause other traders (frontrunners?) to update their own trades,
and if they are malicious, engineer a delayed or hastened match.

### Segregated Mempools for Bidding

The disadvantage of this mechanism is that it is slow and incurs transaction fees both to
make bids and to revoke them. The alternative is another bid format that avoids on-chain
confirmation until the market-clearing transaction goes through (but still requires 
revocations to be confirmed on chain). In this case the trader's
bid transaction consumes the UTXO containing the $X$ A tokens and creates a new UTXO
consisting of $Y$ B tokens spendable by the trader himself. The lock script for this
transaction is valid, but the type scripts are not because B tokens are conjured out of
thin air. So this transaction cannot be confirmed unless it is merged with opposite bids.

As long as no order book operator has spent this UTXO, the trader can cancel the bid 
by spending the UTXO of the A tokens to himself. Whenever a market clearing transaction 
occurs at a compatible price, this transaction can be merged into it. At this point the 
type scripts will pass and the order book operator can generate a proof for that. The
bids need to circulate in a segregated mempool shared among order book operators because
Neptune nodes that are not order book operators do not accept invalid transactions and
will punish peers that announce them.

Segregated bidding mempools ultimately suffer from the same drawback as on-chain bidding
does: the unknown variable is the time before the order closes or whether it closes at
all. Moreover, the time before it is executed gives other traders a window of opportunity
to update their own bids, whether maliciously or not. While the capability to update one's
bid one presents the trader with more flexibility in adapting to market conditions, it
benefits the frontrunners as well.

### Lightning

The disadvantage of the above mechanism is that trades are still rather slow because the
market clearing transaction needs to be confirmed. If instead of waiting for on-chain
confirmation, traders have Lightning channels either between themselves and with the order
book operator, then the market can be cleared through Lightning transactions.

To make a bid in a Lightning-based order book, the trader routes a single payment that
debits his balance with $X$ number of A tokens and credits it with $Y$ number of B tokens.
The payment is conditional upon the release of a preimage known only to the order book 
operator. In order to effect a market-clearing transaction, the order book operator 
releases the preimages of all bids involved. Every participant in all the hops of the 
entire payment now has to update their channel balances off-chain or risk losing
the on-chain disputation process.

The benefit of Lightning-based order books is twofold: (a) arbitrary thoughput capacity,
since one transaction no longer competes with another for scarce blockchain space; and
(b) lightning-speed settlement, which in the present context translates to
*decentralized high-frequency trading*. The disadvantages are that they require (a)
capital to be locked up in Lightning channels, and (b) bids to be tailored to the order
book – making different order books exclusionary marketplaces and reinforcing the
centralization paradox.

Lightning-based order books constitute a comprehensive deterrence to frontrunning because
the attacker's window of opportunity is shrunk to the tiniest possible duration. It is
limited not by the consensus mechanism but by the speed of computation and the propagation
delay of packets on the internet.

## Another Look at Automation

From the point of view of the end-user the experience of automatic markets is no different
from one that is operated behind the scenes. He presses the "OK" button on a trade and it
goes through. Given enough capital, the presence of operators is virtually guaranteed and
with it the user experience of automation.

The differentiating factor between the order books described here and automatic market
makers is *not* the automatic user experience but instead the mechanism that incentivizes
the presence of liquidity. Specifically, liquidity providers are rewarded from day one
for storing their wealth in the liquidity pool. In contrast, for the order books described
here the profit comes not from sitting on liquidity but from matching orders. As a result,
there is no incentive for liquidity providers to park their wealth there. Consequently,
order books face a chicken and egg problem. Without trading activity, it's difficult to
estimate how long your order might take to be executed. So why bother placing the first
bid? From this point of view, MEV in exchange for a self-bootstrapping mechanism is a
favorable tradeoff.

## Conclusion

As trading evolves, rational traders migrate to platforms whenever they can save costs by
doing so. Neptune is well suited to be the meta-platform on which this journey will
continue.

 - Neptune can support a two-way peg from Ethereum for ERC-20 tokens, which will make
   their trade on Neptune instead of Ethereum frictionless.
 - Neptune supports confidential transactions for thusly wrapped tokens, which are
   preferable for non-trading transactions and holding.
 - For trading purposes, confidential wrapped ERC-20 trades are impervious to frontrunning
   because they are undetectable. Fora and bid-feeds enable traders to connect with
   counterparties and do not need to leak trade volume.
 - On-chain order books automatically discover price and have a weaker frontrunning
   profile relative to automatic market makers. The reason why on-chain order books
   on Neptune are preferable to those on Ethereum (where they are also possible) is
   because the trader's portfolio is not exposed. After completing a trade, the obtained
   tokens are withdrawn into a private wallet.
 - Segregated mempools for bids accelerate price discovery relative to on-chain order
   books as a result of the option to update bids in one direction off-chain. The faster
   price discovery leads to less slippage.
 - Lightning-based order books represent the next and surely also the last stage of this
   evolutionary process. Bids can be made and updated in both directions at the speed of
   light, or of whichever medium carries them. The price is discovered as quickly as the
   laws of nature permit. The price is discovered here first so eventually all trading
   will migrate here. The fast bidding and clearing is also what provides the optimal
   deterrence against frontrunning attacks. Moreover, Lightning-based order books have
   the positive externality of incentivizing order book operators to become key
   infrastructure providers to the Lightning network.

In the end, it's not transaction confidentiality per se that kills frontrunning attacks 
(although it does help). Instead, frontrunning and MEV are the necessary tradeoff that
come with bootstrapping the decentralized finance ecosystem. As this ecosystem evolves,
the privacy provided by Neptune might be the feature that drives volume there; and the
UTXO and transaction logic of Neptune supports the next stages of its evolution.
Eventually the scalable architecture of Lightning is the kingmaker.

| ![An order book based on Lightning](lightning-based-order-book.png) |
|:--:|
| Fig.1: A Lightning-based order book, by Midjourney |

[^1]: At time of writing, [Flashbots](https://transparency.flashbots.net/) indicates that 28061 Eth was lost to MEV in the last
      30 days. The current price is 2085 USD. So 28061 × 2085 ≈ 600 million USD.
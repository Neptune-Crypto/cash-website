+++
title = "Only Privacy Coins are Fungible"

[extra]
author = "Thorkil Værge"
ogdescription = "Privacy not only benefits the individual user or company. Privacy is crucial to a fundamental property of money: fungibility"
ogimage = "lady-justice-statue.png"
ogtype = "article"
+++

# Only Privacy Coins are Fungible
All other things equal, users would prefer privacy coins over see-through coins like Bitcoin or Ethereum. The reason is quite simple: privacy coins allow users to make their transaction history public but see-through coins do not allow users to keep their transactions private.

What might be less obvious, is that privacy *not only* benefits the individual user from oppressive regimes or common thieves, or the business from industrial espionage: Privacy is crucial to a fundamental property of money: *fungibility*.

## Fungibility
Fungibility is the property of money that equates one unit of money with another. If you lent me two 50-bills of your local currency, and I paid you back with one 100-bill, you would probably consider the debt paid. But if I borrowed your car and gave you back a five years older car, there would be trouble. Currency is fungible, cars and houses are not.

If we used small non-standardized gold nuggets, as was the case in Ancient Egypt, the money would clearly not be fungible and each shopkeeper and private individual would have to spend time assessing the value of the nugget that they were receiving.

But apart from physical differences, there is another way in which fungibility can be destroyed: If a shopkeeper had to know the history of each received banknote to ensure that it hadn't been involved in any theft, drug trade, or other illegal activity, and that the banknotes could get confiscated if that were the case. Or even worse, one could imagine that new information could arise tainting banknotes that were already in possession of the shopkeeper.

## Western Legal Tradition and Fungibility
![](lady-justice-statue.png)
<center>Lady Justice recognizes the importance of the fungibility of banknotes</center></br>

The necessity of history-blind banknotes was discovered early on: In 1748, 28 years before the American Revolution, merchants and bankers in Scotland ran into this very problem: A merchant named Hew Crawfurd had noted the serial number of banknotes belonging to him. When the banknotes got lost in the mail and one of the bills later showed up in the Royal Bank of Scotland, he demanded it returned, as anyone would with stolen goods. As one cannot acquire ownership through a thief, Hew Crawfurd's lawyers argued that the case was simple, that the money should be returned to their rightful owner.

The bank, however, disagreed. Their defense was centered on more practical grounds: if banknotes received in good faith were not the property of the recipient, both banks and banknotes might as well be abolished as it would open the doors for frauds by malicious and conniving persons who would buy goods with their banknotes and later report the same banknotes stolen, a double-spending attack on the currency system. It was a necessary quality, the bank argued, that the history of a banknote could not be used to challenge ownership, provided that the last transaction was done in good faith by the recipient.

![](hew-crawfurd-presenting-his-evidence-to-the-judge.png)
<center>Hew Crawfurd making his case to the courth, as imagined by Midjourney</center></br>

The court case was decided in favor of the bank: If Crawfurd was able to vindicate the banknote, no merchant could risk taking money in payment "without being informed of the whole history of it from the time that it first issued out of the bank or the mint till it came to his hand, which is so apparently absurd, that it seems hardly to merit a consideration".[^kennethreid] This court ruling seems to have been mainly justified from a practical perspective: any other conclusion would stop all trade. Later, both English and US courts have reached the same conclusion.

What then is the problem with see-through blockchains? Sure, the history might reveal problematic transactions for coins in your possession but if the courts agree that money accepted in good faith constitutes the transfer of ownership, why do we need privacy coins?

To this skepticism I offer three answers:
1. The courts of the 1700s may have been more accomodating to the needs of business than present day courts. And present day courts might be more accomodating to the interests of currency issued by their own institutions than a currency developed in anonymity or outside of the control of their government. In fact, an American court has already found that Bitcoin, as oppossed to US banknotes, received in good faith can be confiscated, and that Bitcoin are thus not offered the same legal protection as banknotes are.[^unitedstatesvsbitcoin]
2. Even if a court decides to make see-through coins fungible by ruling so, that ruling holds only until it is overturned. In other words, the holding risk remains.
3. More fundamentally: Is it in the nature of blockchain to rely on benign and well-behaving institutions, or is it rather to take matters into our own hands and offer a protocol solution to a political problem? Just like Bitcoin fixed the problem of the unpredictable and ever-increasing money supply without an appeal to politics, so too must we fix the problem of fungibility.

## Privacy and Fungibility
Introducing privacy at the base layer of a blockchain eliminates the risk for a shopkeeper, bank, pension fund, or private individual that the transaction they receive can be reversed through legal means. The importance of non-reversibility is recognized by Satoshi Nakamoto in the opening paragraph of the [Bitcoin Whitepaper](https://bitcoin.org/bitcoin.pdf). Here, he references the problematic situation prior to Bitcoin's launch:
> Completely non-reversible transactions are not really possible, since financial institutions cannot
avoid mediating disputes. The cost of mediation increases transaction costs, limiting the
minimum practical transaction size and cutting off the possibility for small casual transactions,
and there is a broader cost in the loss of ability to make non-reversible payments for non-
reversible services. With the possibility of reversal, the need for trust spreads. Merchants must
be wary of their customers, hassling them for more information than they would otherwise need.
A certain percentage of fraud is accepted as unavoidable

Satoshi Nakamoto is here referring to the possibility of chargebacks on credit cards and later on correctly (and brilliantly) goes on to identify the proof-of-work solution as a fix to prevent transaction reversals. His fix, however, is not a full solution as a risk of a transaction reversal is not completely eliminated in a world where courts and political systems claim jurisdiction over the blockchain.

In a world where anonymous parties trade with each other online, perhaps the courts would not be able to threaten the fungibility of crypto currencies, but that is not the world we live in. Rather, Bitcoin investors are often clearly identified individuals, companies, or institutions who must obey court orders. If the court demands the reversal of a transactions, they would be wise to comply.

So perhaps Satoshi Nakamoto only came up with a half solution to the problem of reversible transactions, perhaps privacy blockchains are needed to completely eliminate the risk of transaction reversals?

## But what about Recovering Stolen Funds?
It's worth addressing a question that might be lingering: isn't it a benefit that a thief is prevented from spending their ill-gotten currency? Of course it is. But at what cost? Western courts of the 18th century found that the trust we put in paper currency is more important than attempts at preventing the use of ill-gotten gains; the free flow of trade brings greater benefit to society than recruiting all of commerce to chase a few thieves.

That moves the effort of fighting theft to its genesis: The thief must be prevented from getting possession of money that isn't theirs.

This approach echoes a trend in software engineering from the past decade: IT companies prioritize their infrastructure's security against attacks, rather than depend on law enforcement to chase down infiltrators. As commerce increasingly relies on privacy-preserving blockchains, this strategy will only grow in significance.

## Neptune
Neptune is built with this in mind: To truly eliminate risks when receiving or holding crypto currencies, the blockchain must be private; the transaction history of each coin must be hidden such that any legal claim for the reversal of a transaction is voided. Previous attempts at building privacy-preserving blockchains have not reached their full potential, we believe, [because they are not scalable](../scalable-privacy-problem/). Modern cryptography can solve this shortcoming. In particular, the combination of a [zero-knowledge STARK virtual machine](../announcing-tvm) and [mutator sets](../alphanet/) will make it impossible to link inputs and outputs in a transaction and it will do so without demanding an unreasonable computational workload from the network participants.

[^kennethreid]: Prof Kenneth G C Reid [*Banknotes and Their Vindication in Eighteenth-Century Scotland*, University of Edinburgh School of Law Research Paper Series No 2013/19](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2260952). Note that <i>The Royal Bank of Scotland</i> also went under the name The New Bank and can trace its roots back to 1707. Its full name was and is "The Royal Bank of Scotland". It's, to this day, a separate bank from "Bank of Scotland" which also went under the name "The Old Bank" in the 18th century. The suit was against the Royal Bank which was where the banknote entered the banking system. But both banks collaborated in the courtcase as their interests were aligned.

[^unitedstatesvsbitcoin]: This article from Georgetown Law Technology Review suggests precisely this, that US courts will absolve government currencies from holding risk but not crypto currencies: "Thus, it seems unlikely that a court would apply the same property rules relating to money to cryptocurrencies, which—under the common law as codified in the UCC — are not money". This holding risk has already been established for Bitcoin in the court case *United States v. 50.44 Bitcoins*:  [Andrew Balthazor, THE BONA FIDE ACQUISITION RULE APPLIED TO CRYPTOCURRENCY, 3 GEO. L. TECH. REV. 402 (2019)](https://georgetownlawtechreview.org/wp-content/uploads/2019/05/3.1-Balthazor-pp-402-425.pdf)

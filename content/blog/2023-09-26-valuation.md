+++
title = "On the Valuation of Cryptocurrencies"

[extra]
author = "Alan Szepieniec"
ogdescription = "On the Valuation of Cryptocurrencies"
ogimage = "take-my-money.jpg"
ogtype = "article"
+++

# On the Valuation of Cryptocurrencies

How do we determine the right unit price for a cryptocurrency? This article explores some thoughts about wrong and right ways to value cryptocurrencies in general, and Neptune in particular. Nothing in this note constitutes investment advice.

## The Prevailing Strategy

The prevailing strategy for establishing an appropriate unit price for a cryptocurrency uses the fully diluted valuation (FDV), which is the product of the supply limit and the unit price, as a benchmark for comparison. The calculation goes something like this.
 
 - Estimate the rank on coinmarketcap.com or coingecko.com that the cryptocurrency could reasonably occupy based on features such as the quality of its technology, the charisma of the founders, the skills of the developers, the enthusiasm of the community, *etc*. These websites rank order cryptocurrencies by market capitalization, so from this rank one can infer a fully diluted valuation.
 - Discount this fully diluted valuation by an estimation of the probability of the events that need to happen in order for the cryptocurrency to occupy that rank – such as a successful launch, adoption by a major financial institution, a listing on important exchanges, *etc*.
 - Divide the discounted fully diluted valuation by the token supply limit, and *viólà!* you have your unit price.

Even though it sounds intuitively right, I think it is flawed – and it is certainly at odds with how we think about Neptune. Here are some criticisms:

**1. Infinite FDV.** Not all cryptocurrencies have a supply limit, such as Monero and Ethereum. Consequently, their fully diluted valuation is infinite. 

In the interest of steelmanning the opposing argument, one can replace the phrase *fully diluted*, which implicitly looks infinitely far into the future regardless of the inflation or issuance schedule, by *circulating supply at reference time* for some arbitrary but fixed reference point, say 10 years in the future. This *reference time circulating supply valuation (RTCSV)* metric is less catchy but certainly less absurd.

**2. Equity is not Money.** The term "diluted" betrays the origin of the valuation strategy: stocks. An owner of shares in a company is *diluted* if that company issues new shares to other investors. A cryptocurrency holder is *not* being diluted when the protocol inflates the supply *in accordance with a schedule that limits the total supply to a known upper limit*. Instead of computing his share relative to the current circulating supply, the cryptocurrency holder should calculate it relative to this upper limit.

More importantly, it is possible to compute a valuation for a company independently from its stock price, namely through monetizing the assets that the company owns and the future profits it can make. The stock price tends to follow this independent valuation because otherwise an arbitrage opportunity exists. In contrast, cryptocurrencies – *currencies* in general – are valued purely in relation to the exchange rate they are expected to command in the future. As a result, currencies have a much greater upward as well as downward potential relative to traditional stock.

**3. Premines Matter.** The FDV and the prevailing valuation strategy do not take the presence or size of a premine into account. This lack of accounting implicitly asserts that the presence or size of a premine is irrelevant to the market price that a cryptocurrency can command. This hypothesis is obviously untestable for lack of counterfactual, but there are potential reasons worth considering pointing to its falsehood.

 - *The order of events:* Bitcoin came first and certainly drove awareness about cryptocurrencies. If a premined cryptocurrency had come first, cryptocurrencies might never have taken off to begin with due to lack of interest.
 - *Current status quo:* Bitcoin dominates the ranking.
 - *Ideological commitment:* ideologues generate economic demand and technical support independent of market conditions precisely because they are irrational (in the economic sense). A grassroots ideological community is unlikely to support a cryptocurrency whose purpose is perceived to be the enrichment of founders and insiders.

**4. Self-Reinforcing.** The prevailing valuation strategy is a good strategy because it is what the rest of the market uses. The problem with this situation is that it is not stable: it does not correct errors, and as a result errors can explode – or vanish. It is fine to use this strategy when everyone else does, but as soon as the prevailing strategy changes, *someone* is left holding the bag.

More fundamentally, if investors make trades based on valuations informed by current markets, how can they expect to make a profit? The expectation of profits is justified only if the investor possesses information that is not yet reflected in the price. It is worth considering from time to time what that information might be.

| ![House always wins](beaver-in-casino.png) |
|:--:|
| Fig.1: The casino knows that the house always wins. Imagination by Midjourney. |

## So What's Wrong?

The criticisms above provoke thought but are logically unsatisfying because they address the conclusion and not the argument itself. So what part of it is wrong? To be honest, I'm not 100% sure but here are some possibilities.

 - The premise is that the ranking on leaderboards such as coinmarketcap and coingecko is somewhat stable and subject to slow change, if any. In practice, prices jump suddenly and are slowed only by market makers with limited capital at stake.
 - Cryptocurrencies are measured by the wrong set of features, or the priority within this set is wrong. For instance, I would argue that the size of the premine ought to be included. Whatever features can be distilled out of the apparent success of Dogecoin and Shiba Inu should be not be included – that is, assuming (perhaps incorrectly) that these coins are fads destined to fade with time.
 - Valuation of the total supply, whether the asymptotical limit or the circulating supply, is somewhat disconnected from price. Prices are set on the margin. Total supplies affect prices indirectly by informing actors' marginal preferences.

## ~~A Better~~ Another Valuation Strategy

### Potential

An argument for an alternative and hopefully better valuation strategy needs to start with recognizing the true potential of cryptocurrencies, which is (in my estimation) far beyond what the current market would suggest. Cryptocurrencies are not just another set of colored chips at the global roulette table without any features beyond the mediation of gambling. Cryptocurrencies are profoundly disruptive; they will induce lasting changes to global finance and even geopolitics.

Money lies at the heart of every economy. It synthesizes information about supply and demand through the pricing system and consequently coordinates the distribution of labor and resources. It enables building long chains of capital processes that ultimately lift people out of the grinding poverty they they would otherwise endure as anything beyond subsistence farming requires large-scale coordination.

And yet, today's money is inherently political. Legal tender laws recognize only the currency of the state and exclude competitors. Except for a handful of micronations, every country on earth has a central bank in charge of the currency supply. The central bankers are selected not through election or transparent meritocracy but appointed by the political class. The central banks, together with the financial regulators (another *de facto* branch of the government pretending to be independent of politics), dictate the rules that banks need to follow if they want to keep their license and insurance against bank runs. Large transactions in cash are being phased out and as a result the only way to make large expenditures is with a bank account. To open such a bank account, a statei-issued identity token is required, not to mention the sacrifice of one's financial privacy and independence, which can be and routinely are violated at political whims. The side effect of political money is the dystopian regime of financial surveillance and control that even Orwell failed to foresee. Its principal objective is to use the violence of the state to protect the banking class from the market forces that would otherwise tear them down. *Fiat money is the tool used by the banking class to socialize their losses while privatizing their profits.*

Cryptocurrency is apolitical money, like gold was. It does not need the blessing or the resources of the state. Demand for them comes from the market, not from a state edict requiring the payment of taxes in kind. They are designed to store and transfer wealth, not to hide the wealth redistribution that is taking place against the owners' will. Participation is voluntary for all parties to an exchange. As such, it does not only offer an antidote to the dystopian regime of financial surveillance and control, but it also offers a counterweight to the excesses of the banking class.

Individual emancipation and monetary restraint paint only half of the picture. The other half comes from a geopolitical perspective. Since fiat money is inherently political, it is used as a cudgel in geopolitical conflicts. The regime of surveillance and control over the financial system gives the state the unique ability to gatekeep access to the economy. We see evidence of the weaponization of money in

 - the stockpiling of foreign treasury bonds for leverage;
 - the sanctions imposed on states, companies, NGOs, and even individuals that are deemed hostile;
 - nation-wide bans from SWIFT;
 - the seizure of foreign exchange assets;
 - the designationg of activities that circumvent the financial grid as money laundering, in the absence of evidence of crimes.

For better or for worse, cryptocurrencies stand to deprive the state of these tools of soft war. International commerce will be driven to cryptocurrencies as a result of its desire to avoid getting caught in the crossfire. Due to their geopolitical neutrality, *cryptocurrencies represent the new Schelling point for global finance*.

To assess the true potential of cryptocurrencies one must take into account their effects on individual emancipation, country-wide monetary policy, and geopolitics. In this frame, the current market capitalization of 1 trillion CHF is a blip; even the market capitalization of gold at 14 trillion CHF is barely significant. The *fiat currency* market is 100 trillion CHF. At this point one must question the wisdom of denominating the value of the cryptocurrency market in something that it is hypothesized to diplace. Does *any* price tag make sense?

It is worth noting also that cryptocurrencies do not merely compete with fiat currencies. By design, fiat currencies are poor media to store wealth across large spans of time. The inadequacy of fiat for this purpose is a large factor driving demand for real estate (4-50 trillion CHF depending on estimates) and bond markets (133 trillion CHF). Likewise, cryptocurrencies offer the possibility of executing arbitrarily complex contracts at the speed of lightning, *without trusted intermediaries*. It seems inevitable that cryptocurrencies stand to absorb a good chunk of the global derivatives market (500 trillion CHF) as a consequence.

| ![The scene in the temple before the busy beaver of cryptocurrency turns the table.](beavers-money-changers.png) |
|:--:|
| Fig.2: The scene in the temple before the turning of the tables, as imagined by Midjourney. |

### Success Factors

Having decided on a number, the next step is to estimate the probability of success. There are two separate families of events to consider here. 
 1. The sequences of events sufficient for cryptocurrencies as an asset class to attain the macroeconomically and geopolitically relevant status asserted above. 
 2. The sequences of events sufficient for Neptune in particular to emerge as the dominant option within the set of all cryptocurrencies (assuming the objective is to calculate a valuation for Neptune).

My main objection to the prevailing strategy for valuation is that it restricts attention to the second family of events while ignoring the first. In reality, they are intertwined and should not be assessed separately. The context of a struggle between the financial surveillance and control state on the one hand, and free individual and international commerce on the other, is precisely where Neptune shines and where its relative advantage over competing cryptocurrencies is maximized.

Specifically, the state stands to lose power as cryptocurrencies become more popular. The response may be efforts directed towards eliminating this threat or towards subsuming it as part of the state's financial surveillance and control system. One of the obvious and under-asked questions is, how resilient is the cryptocurrency against state countermeasures? Clearly, any cryptocurrency that can be taken down by a state will never reach geopolitical significance.

In this line of reasoning, the following questions are relevant.

**1. Governance.** Any central point of failure is an obvious target, and even protocols with relatively few central points of simultaneous failure are vulnerable. Permissionless cryptocurrencies generally do a good job (with some exceptions) distributing the necessary compute work, but what they often fail to do is decentralize the protocol governance. Who decides how the protocol is updated? Who pays them, what jurisdictions do they live in, and what are their squeezable pendula?

One option is for the protocol to ossify, in which case the answer is *no-one*. This option is risky though because it might happen that a change is necessary for survival, and in this case the ossified protocol will die.

A second option is for the answer to be vague and ever-changing. In this case, if a contributor with decision-making influence can be identified, it is unlikely that putting pressure on them is possible or likely to change the outcome of whatever "upgrade" is proposed.

The third option is the option we don't want: the answer is clear and definite. It gives the state a target for corruption, co-option, or plain pressure.

**2. Ideological Commitment.** Without a foundation to pay their wages, why would developers contribute to the maintenance and possible upgrades of a protocol? Charities or the direct labor of volunteers can do the job, but only if the project's values align with the charities' and volunteers' personal moral codes. What has the project done to earn this ideological commitment?

**3. Effectiveness of Bans and Orders.** Suppose the state bans certain practices, or decrees that those practices are performed in a certain "compliant" way. What incentives do users have to obey those directives? A well-designed cryptocurrency makes civil disobedience cheap. Smart states will deliberate carefully before issuing bans and ordinances they know in advance to be ineffective.

On a related note, I've argued [previously](../2022-05-20-proof-of-stake-is-not-objective) that the inability of proof-of-stake consensus mechanisms to converge on a single canonical branch if a fork arises, will inevitably lead to state courts deciding canonicity. From the perspective of a state seeking to undermine or co-opt the cryptocurrency, this necessity of court rulings seems like an obvious entrypoint for attack. The strategy is simple: consistently prefer – and have this preference enforced by the justice system – those forks that comply with the law. The end result is complete regulatory capture.

**4. Legal Ownership.** On the topic of legal questions, one of the more nefarious attack vectors is the basic adherence to the long-established legal principle that stolen coins ought to be returned to their proper owner. The obvious proviso of this principle is that the stolen coins can be distinguished from other coins; in other words, the claimant has to prove that the coins in question really are the same coins that were stolen from him.

If the courts do adhere to this principle, it induces a *holding risk*. Individuals and corporations that possess a collection of cryptocurrency tokens might not be the owners of it in the eyes of the law. Furthermore, the information that exposes this difference might come to light after receiving the tokens in exchange for a good or service that cannot be recovered. More maliciously, this principle exposes cryptocurrency holders to legal attack vectors whereby malicious claimants litigate in the hopes of getting a favorable settlement. To protect against this risk, would-be receivers of cryptocurrency need to carefully scan their customers' and their coins' history to rule out illicit behavior.

The alternative would be beneficial to traceable cryptocurrencies but detrimental to politics. Those courts that fail to adhere to this principle and rule instead that coins belong to whoever bears them, risk transforming their jurisdiction into a pariah laundromat for stolen coins.

As far as I can tell there are two solutions this problem. 

 1. A sufficiently large quorum of the world's courts rule in unison that cryptocurrency coins are bearer instruments, and belong to whoever is in possession of them.
 2. The cryptocurrency ensures privacy at the protocol level, thereby [voiding](../only-privacy-coins-are-fungible) such legal attack vectors and eliminating the associated holding risk.

**5. Cryptographic Security.** If there is a cryptographic vulnerability, it will be exploited sooner or later – by hackers if not by state actors. It is one thing to rely on cryptography that is conjectured to be secure by today's experts. But it is another thing to rely on cryptography known to become insecure in the future.

The security level is often set to the surprisingly small numbers on account of efficiency, sometimes as low as 100 or even 90 bits. (For comparison Bitcoin has 94.4 bits worth of cumulative proof-of-work to date.) Across how many iterations of [Moore's law](https://en.wikipedia.org/wiki/Moore%27s_law) do market players want to preserve wealth?

Likewise, universal quantum computers are known to be able to break elliptic curve based digital signature algorithms. It is not inconceivable to assume that protocols such as Bitcoin and Ethereum will receive an upgrade to enable post-quantum signature schemes. What is inconceivable, though, is that *all* addresses currently tied to quantumly-insecure coins will be either upgraded or burned. If this does not happen, the coins they unlock will be up for grabs to the winner in the race to build quantum computers; and if this winner does not cash out, the runner-up will.

**6. Congestion.** Assume the cryptocurrency does become popular, how stable are its features under the pressure of increased demand?

The property of the network to process a larger transaction volume without increasing the fees is called *throughput scalability*. There is no magic pill for throughput scalability; only good and bad tradeoffs. The strategy that Bitcoin seems to be deploying with the Lightning Network (and the strategy that we endorse here at Neptune) is to sacrifice finality but not security. If your Bitcoins or Neptune coins are stored in a Lightning contract, the process of converting them back into on-chain coins can take a while.

The property of the protocol that makes the cost of participation practically independent of the network's popularity or volume is known as *network scalability*. It can be achieved through clever cryptography such as accumulators and SNARKs. Older protocols missed the window of opportunity to integrate these tools into their consensus logic. As a result, network scalability in these older protocols is achieved only through the labor of support nodes, whereas in protocols whose consensus logic integrates this support this property emerges from the one cryptoeconomic assumption underlying every blockchain, which is that participants follow the rules out of self interest.

| ![Beavers build things out of proportion to their size](beavers-cosmic-construction-workers.png) |
|:--:|
| Fig.3: Laying the foundation for a megastructure of cosmic proportions, as imagined by Midjourney. |

## Numbers

Establishing valuations is very speculative process. This is especially true for currencies, and doubly so when they do not exist yet. Even so, the arguments made in this article would be incomplete without at least a back-of-the-envelope calculation. 

As for the potential target, let's go with the valuation at 100 trillion CHF because it is a round number and because after that point denominating anything in CHF (or any other fiat currency) is silly anyway. 

In the table below I assign according to my own subjective judgment (please plug in your own!), probabilities that the given line of questioning decisively favors Neptune. One can justify using the multiplication operator to aggregate them into one number by pretending that these probabilities are independent events

|    | Line | Pr | Comment |
|----|------|----|---------|
| 1. | Governance | 0.25 | The company behind Neptune does plan to dissolve, but this move is risky: what if there is no-one to pass the torch to? The protocol might fail to adapt even though it is necessary. |
| 2. | Ideological Commitment | 0.1 | Projects are in competition with each other over ideologues and it is hard to compete with Bitcoin due to its [immaculate conception](../the-immaculate-conception-of-bitcoin-and-what-it-means-for-competitors). |
| 3. | Effectiveness of Bans and Orders | 0.2 | Cryptocurrencies are difficult to ban in general, and Neptune not more or less so. Neptune is less liable to regulatory capture as it relies on proof-of-work. |
| 4. | Legal Ownership | 0.5 | Neptune features strong privacy at the protocol level, but in this dimension competes with other privacy coins such as Monero and ZCash. |
| 5. | Cryptographic Security | 0.5 | Neptune uses at least 160 bits everywhere security and promises to resist attacks deployed on future quantum computers. |
| 6. | Congestion | 0.25 | Neptune's consensus logic supports light clients for network scalability. It supports scripting, so a Lightning network is feasible. Moreover these scripts are not re-executed but their proofs are verified by the STARK verifier. As a result, it is easy to aggregate transactions in layer-2 rollups. That said, Bitcoin's Lightning network and Ethereum's rollups do have a head start. |

The product of all these numbers is 0.0003125 * 100 trillion CHF = 31.25 billion CHF. This number is not a standalone valuation but rather an alternative *starting point*. There are plenty of other risk and success factors but conventional valuation strategies already account for those.

| ![Busy beaver number](busy-beaver-number.png) |
|:--:|
| Fig.4: Exploring the [limits](https://en.wikipedia.org/wiki/Busy_beaver) of how big numbers can get; imagined by Midjourney. |
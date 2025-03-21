+++
title = "In Defense of Proof of Work"

[extra]
author = "Thorkil Værge"
ogdescription = "There is a widespread belief that proof-of-work mining is a relic of the past. But if miners always use the cheapest available form of energy, maybe that would be good for both the environment and the power grid?"
ogimage = "giga-energy-flare-mining.jpg"
ogtype = "article"
+++


# In Defense of Proof of Work

## Introduction
There is a widespread belief that proof-of-work mining is a relic of the past, that the future of blockchain lies with its competitor, proof of stake. Here, I will attempt to rebuke some of the criticism that proof-of-work faces and argue why it is here to stay.

In proof-of-work mining, participants compete to perform computations in service of the network, in essence calculating tickets to the blockchain raffle, where the prize is finding the next block and with it a batch of newly minted coins. The faster you can compute, the more tickets you get. The mining usually takes place on specialized hardware and the mining process, like all other computations, consumes energy. One of the main political attacks that are currently being made against proof-of-work blockchains is based on the energy consumed in the mining process. Several estimates put the yearly energy consumption of blockchain mining at around 100 terawatt hours [cnet.com][1] [coindesk.com][2]. To put this into context, this mining process consumes about 15 \% less energy than the country of Norway. This raises two questions:

- Is the high energy consumption a problem for the environment?
- Will this energy consumption and potential environmental impact lead to political intervention?

## Is the high energy consumption a problem for the environment?
### The Current Situation
The significant amount of electricity consumed by the mining process does not in itself emit CO2. Only if the electricity comes from non-renewable sources does the mining cause CO2 emmissions. In the electricity sector as a whole, 27.7 \% of the globally produced electricity comes from renewable sources, according to [BP Statictical Review 2021][3][^bprenewablenote], but for the electricity used to mine Bitcoin, this number is between [59 \%][4] and [74 \%][5]. So although Bitcoin mining consumes a lot of energy, this energy is significantly less CO2 intensive than the average electricity production.

How is this trend expected to develop? Can we expect the energy consumption that goes into mining to become less or more CO2 intensive in the future?

### The Economics
A crypto currency mining operation must spend less money on electricity than the value of the crypto currency that it generates, otherwise the operation runs at a loss and is not economically viable. Since miners are competing against each other to find the next block, they are also competing with each other to have the lowest energy price. If you have a lower electricity price than your competitor, you can profitably add more mining equipment to your operation thus raising the difficulty and the power needed to generate the next block. This will push your competitors with high energy prices into unsustainable loss-making positions. By having a lower energy price than your competitors, your mining operation will be economically viable whereas theirs will not. Through this mechanism mining power will flow from those with expensive electricity to those with cheap electricity.

With this logic, it's easy to answer which electricity forms will dominate proof-of-work mining in the future: the cheapest.

### A Case for Hydro
So what will be the cheapest form of energy? It might be tempting to think of coal, because it for a long time has been one of the cheapest forms of energy. But whereever it is available, hydro electricity is a cheaper way to generate electricity than burning coal. Coal has been used heavily throughout history because it's been the cheapest alternative in many areas of the world, but in places where hydro power is available, that is, in mountainous regions with sufficient precipitation, hydro power has always been the cheapest electricity option (sources: [IEA][6], [Wikipedia][7], [Hydro Review][8]).

When electricity is transported over long distances, you lose part of the energy to heat due to resistance in the wires. This and the cost of infrastructure is the reason all of Europe does not get its power from hydro plants on Greenland, and that the US does not get all their power from solar panels in California; this is the reason that we don't have a global energy grid; this is the reason many households and much of industry will have to rely on fossil fuels for the foreseable future and can't get their electricity from solar power from wherever the sun is currently shining. Fossil fuels for electricity generation can be transported to places where electricity is needed and where hydro power is not available.

### Cheaper than Hydro?
Information does not suffer from this geographical restriction. After all, the first telegraph line connecting Europe to America was laid in 1866 whereas the two continents have yet to be connected with either electricity or gas pipes.

The blocks containing the mining rewards are of course just information and are thus not geographically restricted like energy is. Hence blockchain mining will use not the locally cheapest electricity, but rather the globally cheapest energy because they compete on a global market and not a local one. So we should avoid mining with coal since it's not the globally cheapest energy source.

And there might just be an energy form that's cheaper than hydro power: surplus energy. Hydro power has the benefit that the sluices can be shut, and the water in the reservoir saved for a day when electricity is in higher demand. Sun and wind do not have this flexibility: The energy produced with sun and wind must be used immediately as there is practically no storage capacity on the energy grid: You've got to use it or lose it[^useitorloseit].

So we have an energy source that's cheaper than both coal and hydro power: Energy that would otherwise be wasted from sources where the output cannot be controlled. This energy comes from solar and wind power as well as gas from oil fields where the alternative is to burn the natural gas ([Science Direct][9]). The energy price for these technologies may even [become negative][12].

![](giga-energy-flare-mining.jpg)
**Figure 1:** *The future of mining happens with surplus energy. Here, the company [Giga Energy][10] from Texas is using natural gas that would otherwise be flared to mine Bitcoin.*


## Will this energy consumption and potential environmental impact lead to political intervention?
Over time the electricity used for mining should shift to surplus electricity. This, in turn, should remove the impediment, or excuse, for political powers to ban proof-of-work mining since the mining is no longer a net contributor to CO2 emissions. But we can't know how fast this shift will happen, and we also can't know whether the political system will simply invent another excuse to ban mining. So perhaps a better question might be: What capabilities does the government have to ban mining?

Going with the above reasoning, information can easily cross borders. And since a shutdown of mining in a single country will lower the difficulty for other miners by the proportion that the banned country was mining[^difficultyunderbans], it could be argued that the banned mining will simply move to other countries. Concretely, the miners in the country that is hit with a ban would either manage to move their hardware abroad or the hardware would be destroyed by the government instituting the ban. If the owners manage to move the hardware abroad, either by selling it or relocating it themselves, much of the original mining power should come back online. But even if the hardware is destroyed, mining becomes more profitable in other countries, so this should still lead to increased mining abroad, assuming that the hardware for this mining process can be delivered.

![](bitcoin-mining-difficulty-china-ban.png)
**Figure 2:** *Bitcoin difficulty the last 18 months. The large dip starting in May 2021 and ending in October 2021 shows the effect of the Chinese government's ban on Bitcoin mining. Source: [BTC.com][11].*

This is of course just a theoretical prediction, just logical reasoning. Do we have any data to back it up? Yes, we do!

The Chinese government ran this experiment for us: One of the most powerful governments in the world banned proof-of-work mining in May of 2021, and the result can be seen in Figure 2. Prior to the ban, China was responsible for almost half of global Bitcoin mining. Today their share is close to zero, yet only five months after the ban, the global mining power had recovered. This shows that although government bans can be destructive for the individual miners, it cannot threaten the security of the blockchain. The economic forces of proof-of-work are more powerful than any individual government's political power.

### What about a Global Government Ban?
But what if all governments banded together to ban proof-of-work mining, wouldn't that be an existential crisis?

The problem with that line of reasoning is that it underestimes the reasoning outlined above. Businesses **and** governments alike are susceptible to economic incentives: businesses for turnover, and government for taxes. So the more jurisdictions that ban proof-of-work mining, the more the incentive to mine in other jurisdictions will rise. This incentive affects not just miners in other jurisdictions but also their governments, since these governments can tax the mining operations. A ban in all countries would create an enormous incentive for one country to break the ranks, as all the miners would flock to this country that could then reap the tax rewards.

With Bitcoin being declared legal tender in two countries and a few territories[^bitcoinlegaltender], this political dynamics is all the more powerful and obvious: The political future of multiple government leaders is tied up on the survival of proof-of-work blockchains. This makes coordinated political action against blockchains all the more unenforceable and unlikely.

[^bprenewablenote]: Note that in the BP Statistical Review the Renewables for historical reasons does not include hydro power. For the number of 27.7 \% hydro power has been added to the Renewables category, but nuclear power has not.

[^useitorloseit]: Global storage capacity for electricity can provide 5GW of output whereas the required output averaged over a whole year is 3TW [BP Statictical Review][3]. So energy storage facilities can meet about 0.12 \% of energy needs.

[^difficultyunderbans]: If 50 \% of mining dissapears, the difficulty will drop by 50 %. And all other miners will thus see a 100 \% increase in income.

[^bitcoinlegaltender]: El Salvador declared Bitcoin legal tender in 2021, and the Central African Republic in 2022. The territories of Prospera in Honduras and Madeira in Portugal have also granted Bitcoin special status.



[1]: https://www.cnet.com/personal-finance/crypto/heres-how-much-electricity-it-takes-to-mine-bitcoin-and-why-people-are-worried/ "Here's how much electricity it takes to mine Bitcoin and why people are worried"
[2]: https://www.coindesk.com/business/2021/08/18/how-much-energy-does-bitcoin-use/ "How Much Energy Does Bitcoin Use?"
[3]: http://www.bp.com/statisticalreview
[4]: https://bitcoinminingcouncil.com/q4-bitcoin-mining-council-survey-confirms-sustainable-power-mix-and-technological-efficiency/
[5]: https://coinshares.com/research/bitcoin-mining-network-june-2019
[6]: https://www.iea.org/reports/projected-costs-of-generating-electricity-2020
[7]: https://en.wikipedia.org/wiki/Cost_of_electricity_by_source#Global_studies
[8]: https://www.hydroreview.com/business-finance/hydropower-remains-the-lowest-cost-source-of-electricity-globally/
[9]: https://www.sciencedirect.com/science/article/pii/S2589004222000396
[10]: https://gigaenergy.com
[11]: https://btc.com
[12]: https://www.cleanenergywire.org/factsheets/why-power-prices-turn-negative

# STP - Songbird Test Proposal 1

stp | title | status | type | author | created
--- | --- | --- | --- | --- | ---
1 | Reduce FTSO vote power cap to 5% and enforce stronger randoms | Proposal | Meta | alenabelium | 2021-11-24

## Brief description

Flare Time Series Oracle (FTSO) system is used to obtain time series data (signals) from data providers in a decentralized manner. 
Vote power cap is used to limit the impact of an individual data provider in the final decision for the median price, thus helping to enforce decentralization. Currently vote power cap is set to 10%. As the number of data providers grows steadily, this proposal aims at lowering the vote power cap to 5%. With more growth the cap may be lowered even more in future, but this is a subject of future proposals.

Data providers use a special commit-reveal scheme when sending signals. The scheme facilitates hash functions and random numbers to protect their votes from disclosure during the commit phase. If random numbers are weak, their votes could be disclosed while voting is still in progress. This  would enable other data providers to adapt their signals based on the disclosed votes to get a reward more easily. This proposal addresses the mechanism that prevents usage of small random numbers by enforcing the size of the random number to be at least ??.

## Technical description.

FTSO system is a decentralized time series voting mechanism where every 3 minutes data providers cast their votes for signals. At the moment it supports 11 price signals for various crypto currencies priced in USD. Each 3 minute voting interval is called a *price epoch*. A *data provider* is an entity behind a Songbird account that for each price epoch sends what it deems the correct value for the signal for that price epoch. Signal voting is done by calling certain functions of the smart contracts in FTSO system. During a price epoch a data provider sends a commit hash using `submitPriceHash` function, where it submits the hash of the price, a random number and its address. After price epoch ends and all votes are casted, the next price epoch starts and at the same time the 90s *reveal epoch* starts as well. During the reveal epoch the data behind the hash are revealed and sent by `revealPrice` function to fully cast the vote. 

The quality of a random number used in a commit phase is crucial for good vote protection. If a particular data provider `A` tends to send small random numbers, a malicious data provider `B` could use information about expected price range and the address of `A`, and try to guess the the combination of the price and the random number by pure brute-force trial. In such a case the price could be disclosed before the end of price epoch and used by the malicious data provider `B` to adapt its price signal, in order to increase its chances for getting the reward. Note that a malicious data provider may have only few seconds to carry out such an attack. In any case it is reasonable for the system to enforce stronger random numbers where possible. In addition, the random numbers supplied by data providers are used as a randomess source for use in smart contracts. The proposition aims at rejecting votes with too small random number in reveal phase. Such a rejected vote does not enter into the weighted median calculation. Also, the rejected random number does not contribute to the network randomness source. 

Data providers' votes are weighted. *Balance vote power* of a Songbird account is equal to the balance of the WSGB (wrapped Songbird) token on that account. Accounts on Songbird network can delegate their balance vote power to other accounts. *Vote power* of an account is the sum of the balance vote power of the account and all delegated balance vote powers by other accounts. Data providers' weights in voting are derived from vote power in specific blocks called *vote power blocks*. The period of validity of a particular vote power block is called *reward epoch*. Every 7 days a new reward epoch is estabished automatically and the vote power block is chosen randomly from the past. The vote power of a data provider in the vote power block for a specific reward epoch is deemed *active vote power* for the data provider in the reward epoch. Active vote power of a data provider in a reward epoch is the weight used in voting for that reward epoch.

*Total vote power* is the number of WSGB tokens in circulation. *Active total vote power* for a reward epoch is total vote power in the vote power block for the reward epoch. *Vote power cap* specifies the percentage of active total vote power, to which active vote power of any data provider in the reward epoch is capped. 

*Example 1.* Consider a situation in the current (last) block:
- vote power cap: 10%,
- total vote power: 1234 WSGB,
- vote power of data provider A: 150 WSGB,
- vote power of data provider B: 90 WSGB.

Say the current reward epoch has started 2 days ago and some vote power block was chosen from 3 days ago. In the vote power block the situation was as follows:
- vote power cap 10%,
- total vote power: 1000 WSGB,
- vote power of data provider A: 110 WSGB,
- vote power of dat provider B: 88 WSGB,

Hence:
- vote power cap defines the maximal active vote power of a data provider to be: 10% * 1000 WSGB = 100 WSGB,
- active vote power of data provider A: min(110 WSGB, 100 WSGB) = 100 WSGB
- active vote power of data provider B: min(88 WSGB, 100 WSGB) = 88 WSGB

Active vote powers are used for dual purposes:
- as a voting weight participating in FTSO voting (the weighted median algorithm), and
- as a relative share for reward distributon. 
Namely, in each price epoch the weighted median algorithm calculates the weighted median price and determines which votes (data providers) get the reward. The total available reward for the price epoch is shared between winning votes according to the relative shares of weights.

Thus the impact of the vote power cap is dual as well:
- it limits the power of a data provider in voting and hence deciding the price,
- it limits the rewards that data provider gets.

The latter is particularly important since the amount of rewards a data provider is able to get impacts the yields of delegators. The single most important yield metric for delegators is namely:

```
Reward rate = reward earned in reward epoch / active total vote power * (1 - fee)
```

It indicates the yield of a delegator for invested (delegated) vote power adjusted for the fee. Here *fee* is defined as a fraction value between 0 and 1, where for example 0.2 means 20% fee. 

*Example 2.* A data provider has this situation in a reward epoch:
- active vote power: 100 WSGB, 
- rewards earned: 1 WSGB,
- fee: 20% (0.2).

Then the reward rate equals: `1/100(1-0.2) = 0.008`. This means that for each invested 1 WSGB, the delegator will get 0.008 SGB yield.
Note that in publicly used metrics reward rate is usually multiplied by 100 for easier interpretation: how many SGBs were obtained per delegated 100 WSGB. Note the distinction: delegation and vote power is measured in WSGB, while rewards are delivered in SGB.

Consequently, if the vote power cap is applied to a very successful data provider with too many delegations, this will have a negative effect on its reward rate. This in effect makes the data provider less attractive and helps in lowering its delegations, thus lowering the vote power.
For example, in the case above in *Example 1*, the data provider was capped from 110 WSGB to 100WSGB, hence the "real" active vote power was divided by factor 1.1. The impact of capping in this case has basically the same effect to its reward rate like dividing it by the factor 1.1.

## Link to code repository.

Vote power cap is a governance parameter and changing it does not require any code changes.

WHERE WILL THE RANDOM NUMBER SIZE CHECK BE IMPLEMENTED?

## Link to audit report if applicable.

## Bug bounty information.

## Proposed implementation date range.

## Voting contract details.

## Deadline for voting.
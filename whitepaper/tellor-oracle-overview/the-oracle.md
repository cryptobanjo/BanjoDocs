# The Oracle

Tellor pushes the horizon of the oracle far past arbitrary price data. The Tellor oracle is a protocol for answering on-chain **any question** of **any format**.

At a high level, Tellor is an oracle system where a bonded set of “reporters” answer questions on-chain for others to use freely. To create a properly incentivized system, Tellor mints a native token, “Tributes” (TRB). Rewards in TRB incentivize reporters to submit data using both peer-to-peer payments and inflationary rewards. Using TRB, parties can “tip” a specific question or “query” they want updated, then reporters can choose whether the reward for fetching the data is worth the cost of placing the value on-chain. The security of Tellor comes through a deposit of TRB that acts as a bond or stake requirement in order for reporters to participate in providing data. The reporters risk losing this stake if they submit data that is successfully disputed.

### Data Submission

To become a reporter, an address will deposit 100 TRB. Those TRB are locked (they act as a bond) until the reporter requests to withdraw them. The reporter then must wait one week before they can withdraw.

Once the reporter has submitted their deposit, a reporter can submit values for any query (e.g. ETH/USD, Bitcoin block header information, weather data). After a reporter submits a value however, they must wait a certain time period (a configurable variable, starting at 12 hours), before submitting again. This is both to allow time for disputes as well as to increase the number of reporters in the system.

Reporters can choose to submit values for any ID they want, but in practice will likely pick the ID with the highest tip. ID’s can be updated as frequently as they want. Reporters are rewarded in two ways :

* The tip (half of the tip is burned, the other half goes to the reporter)
* Time-based inflationary rewards

Time-based inflationary rewards are a growing amount of tokens that resets after each mining event. These rewards start at zero and grow at a constant rate of .5 TRB per 5 minutes (this is configurable). When an ID is reported, the time-based rewards go to that reporter and then the amount for the next report restarts at zero. These rewards help ensure liveness by keeping the Tellor system going in times of low demand and in times of higher gas prices, allowing reporters to better predict returns.

![](<../../.gitbook/assets/0 (2)>)

For parties needing data more frequently than when the time-based rewards are greater than the gas costs, they can simply add tips. The Tellor oracle can therefore be as fast as needed, parties will just need to pay for tips to cover expenses of the reporters.

### The Data

Each request for data is given an ID on-chain, but specifications for the data are off-chain. Tellor uses a method for hashing a given query’s specifications and the resulting hash is the bytes32 ID. This flexibility allows anyone to define a data specification and have a method for parsing the data for reporters to read.

Data (returned values from reporters) are submitted as bytes, meaning any data type or number of variables can be pulled in a single query. An example of this type of data could be that a new request ID could be a BTC blockheader, a simple price (e.g. ETH/USD), or even an array of prices (e.g. \[ETH price, BTC price, SPX price, VIX, EUR/USD]). The more data batching the system can provide in a query, the more efficient the system becomes (more values per transaction).

Reporters get to select which values they submit for. Some data requests may be very vague (e.g. the price of BTC/USD), but some may be more specific or manual (the rainfall in inches in Nairobi as measured by one website). Community members can propose (and tip on) new queries under the guide rails of Tellor’s data specifications. These specifications will provide clarity as to data definition for those needing to verify the validity of the data. If reporters do not feel comfortable submitting or supporting a certain query (e.g. if you need a paid api feed), the reporter does not have to submit for it. Parties who wish to build reporter support for their query should follow best practices when selecting data for their query (publish data specification on github, promote/ educate in the community), but will also need to tip a higher amount to incentivize activity.

### Disputes

Tellor data values can be used as soon as the data is placed on-chain, however the longer a user waits once the data is submitted on chain, the more probable it is to remain, and therefore be secure; assuming any value that remains on-chain is valid due to economic incentives to dispute invalid ones. Values are able to be disputed and taken off-chain for the same time frame as the reporter lock ( a configurable variable starting at 12 hours)

![](<../../.gitbook/assets/1 (1)>)

Any party can challenge data submissions when a value is placed on-chain. A challenger must submit a dispute fee to each challenge. Once a challenge is submitted, the potentially malicious reporter (C in Figure 2) who submitted the value is placed in a locked state for the duration of the vote. For the next two days, the Tellor governance contract votes on the validity of the reported value (D in Figure 2). A proper submission is one that corresponds to a valid query as defined off-chain in the Tellor dataSpecs\[1]. Although a correct answer should be known to the miners, the ambiguity (lack of an exact correctness in this case) of validity is a feature and corresponds to “correct” being at the discretion or interpretation of the Tellor community.\[2]

#### Dispute Rounds

The Tellor dispute mechanism allows for multiple rounds of disputes. After the first round, the cost of each subsequent dispute round increases exponentially:

$$
disputeFee_{id,t,r>1} = disputeFee_i  \times 2^{disputeRounds_{id,t} -1}
$$

Where

$$
disputeFee_i
$$

is the initial dispute fee

$$
disputeRounds_{id,t}
$$

is the number of dispute rounds for a specific ID and timestamp

#### Dispute Resolution <a href="#_piarsi92ue00" id="_piarsi92ue00"></a>

At the end of the voting period, and if no new round is initiated, the votes are tallied. If found guilty, the malicious reporter’s deposit goes to the disputing party; otherwise a portion of the fee paid by the disputer is given to the wrongly accused reporter.

#### Dispute Fees

The dispute fee is calculated based upon how many reporters are in the system and for which value you are disputing. The cost to dispute values is:

$$
disputeFee_i = max(10 TRB, bondAmount \times (1 - reporters/200))
$$

Where

$$
bondAmount
$$

is the deposit required from each reporter to be able to provide data

$$
reporters
$$

is the number of bonded reporters that are not under dispute

If multiple disputes are performed on the same ID, a party might be trying to censor values by disputing good values. To counteract this, the dispute fee increases with each dispute on the same ID.

$$
disputeFee_{id,t,r=1} = disputeFee_i \times 2^{disputeCount_{id} - 1}
$$

Where

$$
disputeFee_i
$$

is the initial dispute fee

$$
disputeCount_{id}
$$

is the number of disputes open for a specific ID

For this reason, it quickly becomes prohibitively expensive for a malicious party to simply dispute good values to censor a contract from reading them.

#### Replacement Data Tipping

Once a value is disputed it is taken off chain. For users who do not wish to wait for the result of the two day (or longer) vote, they can simply request the value again. To help support the users in this cost, upon the initiation of the dispute, 10% of the dispute fee is given as a tip to the disputed ID to ensure a quick replacement of the disputed value.

#### Invalid Data Query

Votes can be settled in one of three ways, true, false, or invalid. An invalid result means that the data is removed from the chain, but the reporter and the disputer do not lose tokens. An example of this would be a prediction market on who is the president the day after an election. If the election is not settled (the winner is unknown), but the oracle places a value on-chain before the result is known, it may end up being right, but at the time of the dispute/value submission, it isn’t. This could be a case where the community decides to rule on the dispute as invalid. Leaving this option of ambiguity to the community affords the Tellor system more flexibility and reduces the chances that one of two honest parties (a disputer and a reporter) are punished.

1. [https://github.com/tellor-io/dataSpecs](https://github.com/tellor-io/dataSpecs) ↑
2. [https://medium.com/tellor/subjectivity-in-oracles-f7c3c06f69f1](https://medium.com/tellor/subjectivity-in-oracles-f7c3c06f69f1) ↑
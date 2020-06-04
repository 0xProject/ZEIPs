## Preamble

```
ZEIP: 31
Title: Stake-based Liquidity Incentives
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #31

## Summary

ZEIP-31 introduces a stake-based liquidity incentive to the 0x protocol. This proposal outlines the design principles and constraints of the incentive mechanism, along with guidelines for its implementation. See the official [0x Staking Specification](https://github.com/0xProject/0x-protocol-specification/blob/3.0/staking/staking-specification.md) for a comprehensive explanation of architecture, implementation and usage.

## 1 Overview

### 1.1 Motivation

1. Encourage ownership over the protocol among market makers
2. Reward market makers that stake ZRX tokens
3. Align all market participants with the long-term mission and objectives of 0x
4. Redirect proceeds from arbitrage-driven gas auctions back to market makers

### 1.2 Principles

1. Incentivize liquidity providers to invest in the long-term success of the protocol
2. Create a level playing field across all types of assets and classes of market makers
3. Minimize overhead and cost, both in terms of gas and effort to participate

### 1.3 Specification

1. Participate in governance and liquidity incentives by staking ZRX (section 2.1)
2. Delegate stake to market makers (section 2.2)
3. Funding self-sustaining development with protocol fees (section 2.3)
4. Earn liquidity rewards on trading (section 2.4)
5. Vote on 0x protocol proposals (section 2.5)
6. Scheduling process for claiming rewards (section 2.6)
7. Setting up liquidity rewards for market makers (section 2.7)

## 2 Specification

### 2.1 Staking ZRX

#### 2.1.1 Utility

- Vote on proposals and the allocation of community funds
- Market makers earn liquidity rewards proportional to their trade volume and amount of stake
- Anyone can delegate stake to a market maker to earn a portion of their liquidity reward

#### 2.1.2 Depositing Stake

- Anyone can stake by depositing ZRX into the 0x staking contract
- Stake can be deposited at any time
- ZRX remains staked until withdrawn by the staker

#### 2.1.3 Withdrawing Stake

- Stake can be withdrawn at any time so long as it is not delegated (section 2.2)
- A timelock on withdrawals may be introduced in the future to handle edge cases in on-chain governance

### 2.2 Delegating Stake

#### 2.2.1 Overview

- Market makers can contribute stake on their own behalf
- Third parties can delegate stake to market makers
- Delegated stake has the same utility as regular stake
- Stake contributed by market makers receives additional weight when computing rewards
- A delegator transfers a portion of his voting rights to the market maker (section 2.5)

#### 2.2.2 Incentives

- Market makers can incentivize delegation by sharing a fixed percentage of their liquidity reward with their delegators
- Via delegation market makers can obtain liquidity rewards and voting power without locking up capital in ZRX

#### 2.2.3 Weight of Stake

- Regular stake is always weighted more heavily than delegated stake
- Market makers obtain rewards more cost effectively when staking their own ZRX

#### 2.2.4 Delegating Stake

- ZRX must be staked before it can be delegated
- Anyone can delegate their stake to a market maker at anytime
- Delegating stake comes into affect on the next epoch
- Stake remains delegated until withdrawn by the delegator
- The portion of a market maker's liquidity reward reserved for delegators is held in a reward pool
- Each delegator owns a percentage of the reward pool that is proportional to their stake in the pool

#### 2.2.5 Undelegating Stake

- Stake can be undelegated at any time via a smart contract call; however, stake will remain delegated until the epoch ends
- The delegator continues to earn rewards through to the end of the epoch

### 2.3 Fees

#### 2.3.1 Charging Fees

- Each fill on the 0x Exchange contract incurs a static fee
- The fee scales linearly with the gas price of the transaction
- The fee is denominated in ETH (or WETH)
- The fee is paid in ETH if msg.value is set, otherwise it is taken in WETH
- The fee is paid by whoever submits the transaction
  - This is the taker in an open orderbook relayer model
  - This is the relayer in a matching / closed orderbook model
  - This is an arbitrary third-party when using contract-fillable liquidity (meta-transactions)

#### 2.3.2 Motivation for Fee Mechanics

- Making the fee proportional to the transaction gas price protects market makers from arbitrage-drive gas auctions
- A portion of fees currently paid to miners are recycled back into the 0x ecosystem

#### 2.3.3 Accumulating Fees

- The 0x Exchange contract deposits fees into the staking contract
- The fee is associated with the originating market maker and acts as a measure of their trade volume
- At the end of each epoch the accumulated fees are distributed as liquidity rewards

### 2.4 Liquidity Rewards

#### 2.4.1 Earning Liquidity Rewards

- Liquidity rewards are generated by protocol fees
- Liquidity rewards are paid out at the end of each epoch
- Liquidity rewards are divided between market makers, delegators, and the 0x Ecosystem Fund

#### 2.4.2 Liquidity Rewards for Market Makers

- A market maker's liquidity reward is a function of:
  - The total fees collected across all market makers
  - The amount of stake held by the market maker
  - The amount of stake delegated to the market maker
  - The fees attributed to the market maker
  - A variable subsidy that we add or remove to influence liquidity provision
- 1 Staked ZRX = 1 Reward Credit
- 1 Delegated ZRX = 0.9 Reward Credits

#### 2.4.3 Liquidity Rewards for Delegators

- Delegators receive a portion of the reward for each market maker they have delegated to
- This portion is set once by the market maker when creating the delegation pool
- Each delegator earns a reward proportional to the amount they delegated

#### 2.4.4 Liquidity Rewards for the Ecosystem Fund

- The 0x Ecosystem Fund is a pool of ETH collectively managed by ZRX holders
- A small fixed percentage of the total fees collected are allocated to the fund
- Until the 0x Ecosystem Fund has been created all fees will be paid to market makers and their delegators

### 2.5 Governance

#### 2.5.1 Voting on Proposals

- Anyone can vote on a proposal using their stake, the stake they delegate, or using stake delegated to them
- 1 Staked ZRX = 1 Vote
- 1 Delegated ZRX = 0.5 Votes for the delegator and 0.5 Votes for the delegate

### 2.6 Epochs

#### 2.6.1 Scheduling

- All processes are segmented into time intervals called epochs
- Fees accumulate over an epoch and are paid out at the end of the epoch
- Time-locks for withdrawing stake (and exiting reward pools) are measured in epochs
- All epochs last a fixed minimum time
- ~The duration of an epoch is to be determined, but will likely be 2-4 weeks~
- The duration of an epoch is 10 days

#### 2.6.2 Epoch Finalization

- Epoch T must be finalized before Epoch T+1 can start
- Anyone can finalize an epoch by calling a function in the staking contract that:
  - Computes liquidity reward allocations across market makers
  - Pays market makers and escrows the portion allocated to their delegators (section 2.2)
  - Pays a portion of the liquidity rewards to the 0x Ecosystem Fund
  - Increments the epoch
- Stake and delegated stake automatically carry over to the next epoch

### 2.7 Setting Up Liquidity Rewards for Market Makers

#### 2.7.1 Registering as a Market Maker

- A market maker first creates a staking pool
- The maker then registers their trading addresses with the pool (each address must call into the staking contract)
- Each time a 0x order is filled, the maker address is used to lookup a staking pool
- If the pool has a non-negligible amount of ZRX staked then the fee is attributed to the pool

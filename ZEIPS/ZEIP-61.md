## Preamble

```
ZEIP: 61
Title: Initial Staking Parameters
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #61

## Summary
This proposal outlines the initial parameters for stake-based liquidity incentives. Parameters are upgradeable via a vote by the ZRX token holders. See [ZEIP 31](https://github.com/0xProject/ZEIPs/issues/31) for the original stake-based liquidity incentives proposal, and the [0x Staking Specification](https://github.com/0xProject/0x-protocol-specification/blob/3.0/staking/staking-specification.md) for the most up-to-date information on architecture, implementation and usage.

## Specification

|Parameter|Value|
|--|--|
|α (alpha)|2/3|
|Epoch Length|10 days|
|Minimum Stake|100 ZRX|
|Delegated Stake Weight|90%|
|Protocol Fee Multiplier|150,000|

## Rationale

**α (alpha)**
This is a term in the [Cobb-Douglas formula](https://github.com/0xProject/0x-protocol-specification/blob/3.0/staking/staking-specification.md#62-paying-liquidity-rewards-finalization) that dictates the weight of fees versus stake when computing liquidity rewards. By choosing a weighting of 2/3 we are giving a greater weight to fees over stake.

**Epoch Length**
All processes in the system are segmented into contiguous time intervals, called epochs. Protocol fees accumulate throughout an epoch and are distributed when the epoch ends. Stake remains delegated (or undelegated) for the duration of an epoch.

This parameter is the minimum number of seconds between epochs. Short epochs are open to abuse (like griefing) and long epochs aren’t great for UX.

**Minimum Stake**
This is the minimum amount of stake required for a pool to collect rewards. It should be low enough to mitigate griefing through dust pools, but not so high that it creates a significant barrier to entry.

**Delegated Stake Weight**
We incentivize makers to stake their own ZRX by lowering the weight of ZRX delegated to their pool. This mitigates monopolization of staking pools, as it is always more profitable for a maker to create their own pool than delegate to a centralized pool.

**Protocol Fee Multiplier**
The protocol fee per-fill is this parameter times the transaction gas price. The value chosen is roughly equal to the average gas cost of filling an order on 0x.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

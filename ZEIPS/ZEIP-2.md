## Preamble

    ZEIP: 2
    Title: Atomic Order Matching
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2017-07-07

## Simple Summary

Allow a pair of opposing orders to be matched without upfront capital.

## Abstract

## Motivation

Currently, order matching of limit orders is supported through use of `batchFillOrders` and `batchFillOrKillOrders`. While the net result of filling a pair of opposing orders with these functions is the same as an atomic match, a taker is still required to have sufficient funds to fill the first order in order to complete the transaction. This adds an unnecessary cost of capital and is especially problematic when matching very large orders. A taker without the necessary funds to fill the initial order would be required to break down the match into smaller batches, which is also an inefficient use of gas.

## Specification

Add a `matchOrders` function that will atomically fill valid crossing orders without requiring capital upfront. This will lower the barriers to entry of running a centralized matching engine and of arbitraging across exchanges. The requirements of this function would be:

- Takes 2 orders as input parameters
- `orderA.makerToken` == `orderB.takerToken`
- `orderA.takerToken` == `orderB.makerToken`
- `msg.sender` is a valid taker for both orders
- Prices of both orders cross each other
- Makers of both orders receive amounts specified by orders
- `msg.sender` keeps the difference

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

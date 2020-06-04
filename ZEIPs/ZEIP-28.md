## Preamble

    ZEIP: 28
    Title: Arbitrary Fee Tokens
    Author: @PhABC, 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-03-8

Discussion: #28

## Simple Summary

Allow taker and maker fees to be paid in any asset, not only ZRX.

## Abstract

Add two new asset data fields to the order schema so maker and taker fees can be paid in arbitrary assets.

## Motivation

Currently, most relayers take their fee via a spread instead of accepting fees paid in ZRX. The main reasons for this are better UX, lower cost and because relayers and users do not want to transact using ZRX. This ZEIP proposes to allow maker and taker fees to be paid in any asset, not only ZRX.

This also allows all relayers, not just matching relayers to take percentage fees.

## Specification

Add two new fields to the order schema:

- `makerFeeAssetData`: The fee asset to be paid by the maker on fill.
- `takerFeeAssetData`: The fee asset to be paid by the taker on fill.

In `_settleOrder()` and `_settleMatchedOrders()`, transfer maker and taker fees to the fee recipients _after_ transfering maker and taker assets (in case the main assets would help cover the fees).

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

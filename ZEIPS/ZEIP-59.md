## Preamble

    ZEIP: 59
    Title: Mixed assets in "market" operations
    Author: 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-10-23

Discussion: #59

## Summary
Remove the restriction of all orders sharing the same maker or taker asset data in `marketBuy...()` and `marketSell...()` functions.

## Motivation
The V3 world is evolving into one where the same asset can be represented by different asset data encodings. A good example would be with the new Bridge (#47) proxies, which can exchange any ERC20 token but can also carry arbitrary payloads in its asset data.

## Specification
In V2, we copied the `makerAssetData` (in the case of a `marketBuy...()`) or the `takerAssetData` (in the case of `marketSell...()`) to each subsequent order passed into the market functions. This essentially enforced homogenous asset types across all orders.

Going forward, we will simply no longer perform this copy, and fill orders as usual, continuing
to track and assert the maker/taker asset amounts bought/sold.

## Rationale

Because we no longer enforce homogenous asset types, this places more burden on the taker/interface to ensure that orders are, indeed, compatible or desirable.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

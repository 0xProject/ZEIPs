## Preamble

    ZEIP: 50
    Title: marketBuy/SellOrdersFillOrKill
    Author: 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-08-14

## Summary

Remove the old `marketBuyOrders()` and `marketSellOrders()` Exchange functions in `3.0` in favor of just `marketBuyOrdersNoThrow()` and `marketSellOrdersNoThrow()`. Add `marketBuyOrdersFillOrKill()` and `marketSellOrdersFillOrKill()` which wrap these functions.

## Motivation

Our reasoning for this decision was that the typical user performing a market buy or sell would likely not care if any individual order in the operation fails.

One feature of `marketBuy/SellOrdersNoThrow()` is that it will not fail even if it buys or sells _less_ than the amount of taker or maker asset requested. Thus, it’s useful when you’re simply trying to purchase or liquidate as much of an asset as possible. However, there are cases when more deterministic behavior is desirable, i.e., where it must fill _no less_ than the amount requested or revert. This is the motivation for introducing `marketSellOrdersFillOrKill()` and `marketBuyOrdersFillOrKill()`.

## Specification

The behavior of these new functions would be nearly identical to `marketBuyOrdersNoThrow()` and `marketSellOrdersNoThrow()` but with a final check that the combined fill results indicate that the correct amount of assets were sold or bought:

- In the case of `marketBuyOrdersNoThrow()`:
  - `fillResults.makerAssetFilledAmount >= makerAssetFillAmount`
- In the case of `marketSellOrdersNoThrow()`:
  - `fillResults.takerAssetFilledAmount >= takerAssetFillAmount`

## Implementation

The pseudocode for `marketSellOrdersFillOrKill()` would look like:

```python
def marketSellOrdersFillOrKill(orders, taker_asset_to_sell, signatures):
    # Leaving out checks that orders are homogenous for simplicity.
    fill_results = FillResults()
    for order, signature in orders, signatures:
          fill_results += fillOrderNoThrow(
            order,
            taker_asset_to_sell - fill_results.taker_asset_filled,
            signature
          )
          # Stop if we've met our target
          if fill_results.taker_asset_filled >= taker_asset_to_sell:
            break
    # Revert if we did not spend all of taker_asset_fill_amount
    if fill_results.taker_asset_filled < taker_asset_to_sell:
        revert()
    return fill_results
```

The pseudocode for `marketBuyOrdersFillOrKill()` would look like:

```python
def marketBuyOrdersFillOrKill(orders, maker_asset_to_buy, signatures):
    # Leaving out checks that orders are homogenous for simplicity.
    fill_results = FillResults()
    for order, signature in orders, signatures:
          fill_results += fillOrderNoThrow(
            order,
            # Calculate the amount of taker asset to sell given the
            # amount of remaining maker asset we need.
            (maker_asset_to_buy - fill_results.maker_asset_filled)
                * order.taker_asset_amount / order.maker_asset_amount,
            signature
          )
          # Stop if we've met our target
          if fill_results.maker_asset_filled >= maker_asset_to_buy:
            break
    # Revert if we did not spend all of taker_asset_fill_amount
    if fill_results.maker_asset_filled < maker_asset_to_buy:
        revert()
    return fill_results
```

## Notes/Challenges

- If you preferred the original versions because they asserted that all orders were valid, you can still detect invalid orders (as an EOA) by examining the `Fill` event logs. If an order hash does not appear in the logs, it was either unfillable or you hit your target without needing to fill that order.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

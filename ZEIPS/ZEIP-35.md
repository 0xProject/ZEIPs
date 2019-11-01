## Preamble

```
ZEIP: 35
Title: Permissive Order Cancellations
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #35

## Summary

The current `batchCancelOrders` function is difficult to use because the caller runs the risk of the entire transaction reverting if only a single cancellation fails. This is difficult to prevent because a cancellation may fail for non-deterministic reasons (such as the order being expired). This can be improved by doing a no-op instead of reverting for noncritical errors.

The proposed change would alter the logic of `cancelOrder` and `batchCancelOrders`, but would not affect the interface.

## Motivation

This will improve the reliability of cancelling batches of specific orders.

## Specification

The table below compares how failure scenarios are handled when cancelling an order.

|Scenario|Current Action|Proposed Action|
|--|--|--|
|Reentrancy|Revert|Revert|
|Sender Not Authorized|Revert|Revert|
|Order Expired|Revert|No-op|
|Order Already Cancelled|Revert|No-op|
|Order Already Filled|Revert|No-op|
|Order Invalid|Revert|No-op|

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).





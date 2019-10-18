## Preamble

    ZEIP: 20
    Title: Allow for mass order cancellations based on `salt` value
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2018-01-13

## Simple Summary

Allow traders to very cheaply cancel multiple orders in a single transaction with a single state update.

## Abstract

By using an incrementing number as a salt value, all previous orders which contain smaller salt numbers can be cancelled. This allows Market Makers to effeciently bulk cancel any number of orders with one state update.

## Motivation

Currently, each individual order must be cancelled separately, which can be very expensive if required for a large amount of orders. This can prevent market makers from creating multiple orders for fear of large and sudden market moves.

In addition, if a trader forgets or loses their old orders, it is currently not possible to cancel them. This is potentially dangerous, since there is always a chance that other parties are still storing the orders.

## Specification

Traders SHOULD treat the `salt` order field as a timestamp with arbitrary precision. Then we can add the following functionality:

```
mapping (address => uint256) cancelledBefore;   // mapping of maker => timestamp

function cancelOrdersBefore(uint256 timestamp)
    external
{
    require(cancelledBefore[msg.sender] < timestamp);   // removing this check allows makers to "pause" orders, but adds complexity
    cancelledBefore[msg.sender] = timestamp;
}

function fillOrder(...)
    public
    returns (uint256)
{
    require(order.salt > cancelledBefore[order.maker]);
    // remaining fill logic
}
```

Note that while it is in the best interest of makers to treat an order's `salt` as a timestamp, there is no way to enforce this and it should therefore NOT be relied upon for any other functionality (doing so would create incentives for makers to provide inaccurate timestamps).

It would also be useful to allow makers to cancel orders of specific token pairs, but this may add too much extra complexity.

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

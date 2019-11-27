## Preamble

    ZEIP: 18
    Title: Allow `taker` to be different from `sender` with optional `taker` signature
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2018-01-18

## Simple Summary

Allow fills and other 0x operations to be submitted by a third party (e.g Relayers or contracts).

## Abstract

In the current implementation of the Exchange contract, the taker of an order is always msg.sender. By optionally allowing the signature of the taker to be passed into fillOrder, we can decouple these two addresses and allow for much greater flexibility.

## Motivation

This change would allow for much easier implementations of `Exchange` wrapper contracts. Wrapper contracts are currently impractical because they are required to hold the taker's tokens that will be exchanged. Some basic use cases for wrapper contracts include:

- creating orders that can only be filled by addresses in a whitelist
- creating orders that can only be filled with a signature from another party
- creating custom functions for conditionally filling multiple orders (similar to `batchFillOrders` or `fillOrdersUpTo`)

In addition, this change greatly simplifies logic for relayers using the [matching strategy](https://0xproject.com/wiki#Matching). There is now a clear distinction between the `maker` and `taker` of an order in this model, simplifying logic for calculating fees. Tokens will also be swapped directly from the `maker` to `taker`, which results in fewer intermediate state changes and lower gas costs.

## Specification

For the purposes of this proposal it makes sense to rename the `taker` parameter in the message format to `sender`, since the two are no longer necessarily the same (note: a case can be made for including both `taker` and `sender` parameters in the order message format).

Psuedocode for `fillOrder`:

```
struct Order {
    address maker;
    address sender;
    address makerToken;
    address takerToken;
    address feeRecipient;
    uint256 makerTokenAmount;
    uint256 takerTokenAmount;
    uint256 makerFee;
    uint256 takerFee;
    uint256 expirationTimestampInSec;
    uint8 v;
    bytes32 r;
    bytes32 s;
}

struct Fill {
    uint256 takerTokenFillAmount;
    uint256 salt;
    uint8 v;
    bytes32 r;
    bytes32 s;
}

mapping (bytes32 => uint256) filled;

function fillOrder(Order memory order, Fill memory fill)
    public
    returns (uint256 takerTokenFilledAmount)
{
    require(order.sender == msg.sender || order.sender == address(0));

    address taker;
    bytes32 fillHash;
    bytes32 orderHash = getOrderHash(order);

    if (fill.r == bytes32(0)) {
        taker = msg.sender;
    } else {
        fillHash = keccak256(orderHash, fill.takerTokenFillAmount, fill.salt);
        bytes32 msgHash = keccak256("\x19Ethereum Signed Message:\n32", fillHash);
        taker = ecrecover(msgHash, fill.v, fill.r, fill.s);
    }

    ... // remaining fill logic

    if (fillHash != bytes32(0)) {
        filled[fillHash] = takerTokenFilledAmount;  // the filled mapping must be updated an extra time to prevent replay attacks
    }
}

function getOrderHash(Order memory order)
    internal
    returns (bytes32 orderHash)
{
    orderHash = keccak256(...) // calculate order's hash
}
```

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

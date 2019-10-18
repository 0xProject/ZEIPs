## Preamble

    ZEIP: 7
    Title: On-chain order generation by smart contracts
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2017-09-16

## Simple Summary

Allow contracts to create orders without the use of signatures.

## Abstract

Contracts cannot currently generate orders. With ZEIP-1 contracts are able to generate orders by implementing an `isValidSignature` function. This ZEIP will allow contracts to generate orders on demand and dynamically. This allows a contract to create an order in the transaction lifecycle.

## Motivation

Currently, there is no way for smart contracts to generate orders (see issue #1 ).

## Specification

The following additions can be added to the Exchange contract:

```
mapping(bytes32 => bool) public isOrderApproved;

function approveOrder(
    address[5] orderAddresses,
    uint[6] orderValues)
    public
    returns (bool)
{
    require(orderAddresses[0] == msg.sender);
    bytes32 orderHash = getOrderHash(orderAddresses, orderValues);
    isOrderApproved[orderHash] = true;
    return true;
}
```

Then, when verifying an order signature, we check:

```
require(isValidSignature(...) || isOrderApproved[order.orderHash]);
```

## Rationale

This allows any contract to approve an order on-chain in place of a signature.

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

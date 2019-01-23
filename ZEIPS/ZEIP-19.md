## Preamble

    ZEIP: 19
    Title: Replace `isTransferable` with `fillOrder` variant that uses `delegatecall`
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2018-01-01

## Simple Summary (2 Sentences)

Replace the transfer validity check `isTransferable` with a more succint implementation using `delegatecall`. `isTransferable` has caused issues in the past and is expensive.

## Abstract

The `Exchange` contract's `fillOrder` function currently takes an argument `shouldThrowOnInsufficientBalanceOrAllowance`. If set to `false`, balances and allowances of the `makerToken`, `takerToken`, and ZRX will be checked before attempting any token transfers. If any balances or allowances are insufficient, `fillOrder` will fail gracefully be returning 0 and logging an error. This is intended to be used with any functions that fill multiple orders in a single transaction. If a single fill fails, we do not necessarily want the entire transaction to `throw` or `revert`.

This proposal would remove the `shouldThrowOnInsufficientBalanceOrAllowance` argument and add a function that replicates this functionality by calling the `fillOrder` function with `delegatecall`. In this case, balances and allowances are not checked before attempting a token transfer. However, using `delegatecall` ensures that the the subcall returns false on a failed transfer, rather than reverting the entire transaction.

## Motivation

The implementation of `isTransferable` contains many edge cases, is bug prone, and is a constant point of confusion in the `Exchange` contract. With the addition of the `REVERT` opcode in the Byzantium hard fork, it no longer results in gas savings compared to the transaction reverting. In addition, it is extremely expensive to use (making 8 external calls and copying an extra 384 bytes to memory), and using `delegatecall` is significantly cheaper most of the time (making 1 extra external call and copying 480 extra bytes to memory).

## Specification

```
bytes4 constant public FILL_ORDER_OR_THROW_SELECTOR = bytes4(sha3("fillOrderOrThrow(address[5],uint256[6],uint256,uint8,bytes32,bytes32)"));

function fillOrderOrThrow(
    address[5] orderAddresses,
    uint256[6] orderValues,
    uint256 fillTakerTokenAmount,
    uint8 v,
    bytes32 r,
    bytes32 s)
    public
    returns (uint256 takerTokenFilledAmount)
{
    ... // all fill logic, always reverts on a failed transfer
}

function fillOrderOrReturn(
    address[5] orderAddresses,
    uint256[6] orderValues,
    uint256 fillTakerTokenAmount,
    uint8 v,
    bytes32 r,
    bytes32 s)
    public
    public
    returns (bool success)  // if we want to return the amount filled instead, we have to calculate the orderHash and then check the filled mapping before/after delegatecall
{
    return address(this).delegatecall(
        FILL_ORDER_OR_THROW_SELECTOR,
        orderAddresses,
        orderValues,
        fillTakerTokenAmount,
        v,
        r,
        s
    );
}
```

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

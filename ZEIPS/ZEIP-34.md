## Preamble

```
ZEIP: 34
Title: 0x transaction developer experience improvements
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #34

## Summary

This is a catch-all for smaller developer experience improvements to [0x transactions](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md#transactions). Improvements include:

- Logging an event upon successful transaction execution
- Refactoring `executeTransaction` to take a `ZeroExTransaction` struct as an input
- Adding a `bytes` return value to `executeTransaction` that is equal to the return data of the underlying function call

## Motivation

All of these features should make it easier to work with 0x transactions, both off-chain and at the smart contract level.

## Specification

### TransactionExecuted event

Upon successful execution, `executeTransaction` will log a `TransactionExecution` event.

```
// TransactionExecution event is emitted when a ZeroExTransaction is executed.
event TransactionExecution(
    bytes32 indexed transactionHash
);
```

### New executeTransaction function signature

The inputs and outputs of `executeTransaction` will be modified as follows:

```
struct ZeroExTransaction {
    uint256 salt;                   // Arbitrary number to ensure uniqueness of transaction hash.
    uint256 expirationTimeSeconds;  // Timestamp in seconds at which transaction expires.
    uint256 gasPrice;               // gasPrice that transaction is required to be executed with.
    address signerAddress;          // Address of transaction signer.
    bytes data;                     // AbiV2 encoded calldata.
}

/// @dev Executes an Exchange method call in the context of signer.
/// @param transaction 0x transaction containing salt, signerAddress, and data.
/// @param signature Proof that transaction has been signed by signer.
/// @return ABI encoded return data of the underlying Exchange function call.
function executeTransaction(
    LibZeroExTransaction.ZeroExTransaction memory transaction,
    bytes memory signature
)
    public
    payable
    returns (bytes memory);
```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

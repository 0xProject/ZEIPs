## Preamble

    ZEIP: 33
    Title: New/Consolidated Signature Type(s) and Behavior
    Author: 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-04-19

Discussion: #33

## Simple Summary

Introduce new signature types for orders and  and change validation behavior.

## Abstract

Add more robust order and transaction signature validation with the following callback signature types, which accept an entire `Order` or `ZeroExTransaction` object:
- `Validator` (**replaces existing behavior*)
- `EIP1271Wallet`

Also, *all* signature types (including these) will be checked on every fill. This is in contrast to the behavior in `2.0` where we only validate the signature on the first fill.

## Motivation

In `2.0`, we have `Validator` and `Wallet` signature types, which can only validate against a hash, which means the order/transaction must be somehow registered with the validator contract prior to execution to determine meaningful context. It would be more useful if the validator contract was provided with the complete order/transaction data so they can implement more nuanced and dynamic filtering criteria.

Recent improvements to solidity and `ABIEncoderV2` have made this approach relatively simple to execute.

## Specification

### Restrictions
- Validator contracts will be called via `staticcall()`. If the contract attempts to update state during call, the validation will fail.
- Validator contracts using the `Validator` signature type must be registered in advance via `setSignatureValidatorApproval()` (unchanged from `2.0`). This only has to be done once per validator-signer pair.

### Signature Encoding
The `Validator` signature type is tightly packed with the ordered fields:

```solidity
// Arbitrary data passed to validator as `signature`. 0 or more bytes.
bytes signatureData
// Address of the validator contract. 20 bytes.
address validatorAddress
// Signature type. Always `0x05`. 1 byte.
uint8 signatureType
```

The `EIP1271Wallet` signature type is tightly packed with the following ordered fields. Note that the address of the validator contract is implied as the order maker or transaction signer.

```solidity
// Arbitrary signature data passed to validator as `signature`. 0 or more bytes.
bytes signatureData
// Signature type. Always `0x07`. 1 byte.
uint8 signatureType
```

### Implementation
Contracts validating the `Validator` and `EIP1271` signature types follow the EIP-1271 pattern and must expose the following callback:

```solidity
function isValidSignature(
    // ABI-encoded data associated with the signature.
    bytes calldata data,
    // arbitrary signature bytes passed in top-level call.
    bytes calldata signature
)
    external
    view
    // Should return 0x20c13b0b if the signature is valid.
    returns (bytes4 magicValue);
```

#### EIP-1271 `data` Encoding
There are two ways the `data` parameter for `EIP1271Wallet` and `Validator` signatures can be encoded, depending on what type of data is signed:
- For Orders: `abi.encodeWithSelector(bytes4(0x3efe50c8), Order order, bytes32 orderHash)`
- For ZeroExTransactions: `abi.encodeWithSelector(bytes4(0xde047db4), ZeroExTransaction transaction, bytes32 transactionHash)`

#### Example
Here is a complete (if trivial) implementation that validates an Order signature:
```solidity
pragma solidity ^0.5.9;
pragma experimental ABIEncoderV2;

import "@0x/contracts-utils/LibBytes.sol";

contract MyOrderValidator {

    using LibBytes for bytes;

    // 0x order structure
    struct Order {
        address makerAddress;
        address takerAddress;
        address feeRecipientAddress;
        address senderAddress;
        uint256 makerAssetAmount;
        uint256 takerAssetAmount;
        uint256 makerFee;
        uint256 takerFee;
        uint256 expirationTimeSeconds;
        uint256 salt;
        bytes makerAssetData;
        bytes takerAssetData;
        bytes makerFeeAssetData;
        bytes takerFeeAssetData;
    }

    /// @dev Validate an order signature.
    /// @param data The ABI-encoded order and hash.
    /// @param signature Signature data for the order.
    /// @return magicValue 0x20c13b0b if the signature is valid.
    function isValidSignature(
        bytes calldata data,
        bytes calldata signature
    )
        external
        view
        returns (bytes4 magicValue)
    {
        // Decode the order and hash.
        (Order memory order, bytes32 orderHash) = abi.decode(data.slice(4), (Order, bytes32));
        // Validate the order
        // ...
        magicValue = 0x20c13b0b; // Success
    }
}
```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## Preamble

    ZEIP: 17
    Title: Use order schema hash in signature verification for EIP712 compatibility
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2017-12-20
    Requires: EIP712

## Simple Summary (2 Sentences)

Allow the signer to see 0x order data when signing in supported wallets.

## Abstract

[EIP712](https://github.com/ethereum/EIPs/pull/712) describes a standard for human-readable typed data signing. This would allow the signer of a 0x order to see the exact parameters that they are signing with tools such as Metamask. In order to be compatible with this EIP, we must update how signatures are validated in the `Exchange` contract.

## Motivation

Currently, it is very difficult for a user of 0x to verify that the order that they are signing is exactly what they intended to sign. This adds an extra level of trust between users and relayers, potentially opening users up to scams. Open source [tools](https://github.com/ethfinex/0x-order-verify) exist for verifying orders, but they require users to take extra steps, are more difficult to understand, and are unlikely to gain the same level of traction as functionality that is built into the user's wallet.

## Specification

Currently, signatures are verified with:

```
return signer == ecrecover(
    keccak256("\x19Ethereum Signed Message:\n32", hash),
    v,
    r,
    s
);
```

Instead, we first calculate the hash of the order's schema:

```
bytes32 public constant orderSchemaHash = keccak256(
    "address exchangeContractAddress",
    "address makerAddress",
    "address takerAddress",
    "address makerTokenAddress",
    "address takerTokenAddress",
    "address feeRecipientAddress",
    "uint256 makerTokenAmount",
    "uint256 takerTokenAmount",
    "uint256 makerFeeAmount",
    "uint256 takerFeeAmount",
    "uint256 expirationTimestamp",
    "uint256 salt"
);
```

We then can verify signatures with:

```
return signer == ecrecover(
    keccak256(orderSchemaHash, orderHash),
    v,
    r,
    s
);
```

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

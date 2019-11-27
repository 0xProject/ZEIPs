## Preamble

    ZEIP: 32
    Title: Rich Reverts
    Author: 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-04-19

## Simple Summary

Instead of throwing opaque string reverts, 3.0 contracts will instead throw custom, ABI-encoded "rich" revert types augmented with much more useful parameters.

## Motivation

Reverting with expressive error data will enable developers to create more robust applications built on top of the 0x protocol.

Additionally, this is a critical step towards our goal of becoming a language-agnostic protocol, wherein applications can rely more on the contracts themselves and less on off-chain tooling to validate and debug interactions.

## Specification

#### Encoding

Standard string reverts (à la Soldity's `require()` and `revert()` builtins) are ABI-encoded as a function call with signature `Error(string)`.

For example, a `revert("foobar")` would encode as:

```yaml
# 4-byte function selector (keccak of "Error(string)")
08c379a0
# Offset to the error string calldata.
0000000000000000000000000000000000000000000000000000000000000020
# -> Error string data
    # Length of string (6)
    0000000000000000000000000000000000000000000000000000000000000006
    # String bytes
    0000000000000000000000000000000000000000000000000000666f6f626172
```

Our rich reverts follow the same encoding rules but with varying function signatures.

For example, the rich revert `SignatureError` has the signature:

```solidity
SignatureError(uint8 errorCode, bytes32 hash, address signer, bytes signature)
```

If we construct it with the following values:

```solidity
SignatureError(
    // errorCode
    3,
    // signature hash
    0xa3dcd8f6179b531a8c33b675b700708090d4e94d6f6f4cd9e652239a6225db45,
    // signer
    0x828f817d6612f7b477d66591ff96a9e064bcc98a,
    // signature
    0x010aeaf352d05c6dcf64882760014703432133689f4507cd91e81aaa3b289223
      507bc8cf2629ff3ea8a468013a49b32227900be174575ce135ed2560c236dba6
      8802
)
```

The resulting encoding will be:

```yaml
# 4-byte function selector (keccak of "SignatureError(uint8,bytes32,address,bytes)")
7e5a2318
# errorCode, padded to 32-bytes
0000000000000000000000000000000000000000000000000000000000000003
# Signature hash
a3dcd8f6179b531a8c33b675b700708090d4e94d6f6f4cd9e652239a6225db45
# Offset to signature bytes data
0000000000000000000000000000000000000000000000000000000000000060
# -> Signature bytes data
    # Length of bytes (66)
    0000000000000000000000000000000000000000000000000000000000000042
    # bytes data
    010aeaf352d05c6dcf64882760014703432133689f4507cd91e81aaa3b289223
    507bc8cf2629ff3ea8a468013a49b32227900be174575ce135ed2560c236dba6
    8802
```

#### Decoding

Nodes will only return revert data for `eth_call` operations, though it is possible (but not foolproof) to replay a failed transaction using `eth_call` and a block number.

Furthermore, with geth there is an [issue](https://github.com/ethereum/go-ethereum/issues/19027) where the JSON-RPC response for a successful `eth_call` is indistinguishable from a revert. For this reason, (at minimum) we need to check the leading 4 bytes of the return data against a mapping of known error selectors to detect that a revert actually occurred. This is a little inelegant, but with roughly 4 billion combinations of leading 4 bytes, combined with more rigorous conformance checks, false positives should be extremely rare.

Once the exact error type is detected, ABI decoding tools can be used to extract individual parameter values.

For environments where ABI decoding tools are not exposed/available, we will also be deploying a helper contract that decomposes encoded error bytes into its constituent fields.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## Preamble

```
ZEIP: 37
Title: Allow 0x transactions to be cancelled (or expired)
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #37


## Summary
A signed 0x transaction is currently valid until executed. There is no explicit way to cancel or invalidate a transaction. This ZEIP will discuss 3 different mechanisms accomplishing this.

## Motivation
There are pros and cons to allowing 0x transactions to be cancelled. The initial decision to not allow cancels in 2.0 was actually intentional.

### Argument against cancellation
In many cases, a 0x transaction will be submitted by a user that is not actually the transaction signer (such as a relayer). Allowing the transaction signer to cancel adds an additional griefing vector, where the signer can cancel and force the submitter to waste gas.

### Argument for cancellation
If a taker signs a fill transaction that can only be executed by a different address (i.e when `order.senderAddress` is not the taker), that address can withhold the transaction and submit it at a later time. If the market price has moved during this time, a taker may end up filling an order at an undesirable price.

## Specification
There are at least 3 viable options for allowing 0x transaction cancellations: a `cancelTransaction` function; a monotonically increasing `nonce`; or an `expirationTimeSeconds` field. The three options are described in detail below. *For V3 of the Exchange, we have gone with Option #3: adding an `expirationTimeSeconds` field to transactions*.

### 1 `cancelTransaction` function
The logic for this would be very similar to `cancelOrder`. Pseudocode:

```
function cancelTransaction(ZeroExTransaction memory transaction)
    public
{
    address signerAddress = getCurrentContextAddress();
    require(
        transaction.signerAddress == signerAddress,
        "INVALID_SIGNER"
    );
    bytes32 transactionHash = getTransactionHash(transaction);
    transactions[transactionHash] = true;
}
```

### 2 Force salt to be strictly incrementing by 1
A transaction's `salt` field would serve the same functionality as an Ethereum transaction's `nonce` field (and could also be renamed to reflect this). Valid transactions must have a `nonce` equal to the most recently submitted transaction's `nonce` + 1. Multiple transactions with the same `nonce` cannot be submitted, so a "cancellation" would merely be submitting a transaction with the same nonce as the transaction that is intended to be cancelled.

### 3 Add an `expirationTimeSeconds` field
Transactions mined in a block with timestamp at or after the `expirationTimeSeconds` field would be invalid. This is my personal favorite approach, as it seems to minimize griefing vectors while also limiting the potential damage of a withholding attack (most transactions would likely be very short lived).



## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

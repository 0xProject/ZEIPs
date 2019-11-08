## Preamble

```
ZEIP: 36
Title: Allow batch 0x transaction execution
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #36

## Summary
The Exchange contract currently only allows for a single 0x transaction to be executed atomically. This proposal would add the ability to atomically execute a batch of transactions by calling a single function.

## Motivation
There are niche use cases for composing custom a custom series of Exchange function calls. One example:

1. A maker places a buy order on market A
2. A sell order appears on market B that would satisfy the maker's buy order
3. The maker would like to atomically fill the sell order and cancel the buy order, rather than executing the transactions asynchronously and risking that only one of the transactions is successful.

## Specification
A naive solution is extremely simple to implement and adds no additional attack surface:

```
/// @dev Executes a batch of Exchange method calls in the context of signer(s).
/// @param transactions Array of 0x transactions containing salt, signerAddress, and data.
/// @param signatures Array of proofs that transactions have been signed by signer(s).
/// @return Array containing ABI encoded return data for each of the underlying Exchange function calls.
function batchExecuteTransactions(
    ZeroExTransaction[] memory transactions,
    bytes[] memory signatures
)
    public
    returns (bytes[] memory)
{
    uint256 length = transactions.length;
    bytes[] memory returnData = new bytes[](length);
    for (uint256 i = 0; i != length; i++) {
        returnData[i] = _executeTransaction(transactions[i], signatures[i]);
    }
    return returnData;
}
```

It would be useful if the return data of each transaction could be passed into and processed by the next transaction, but this would be extremely complex to implement (and is potentially infeasible).

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

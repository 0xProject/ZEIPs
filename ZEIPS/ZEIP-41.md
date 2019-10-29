## Preamble

```
ZEIP: 41
Title: Batch Order Matching
Author: 0x Core Team
Type: Standard Track
Category: Core
Status: Final
Created: 2019-10-29
```

Discussion: #41

## Summary

We propose a new feature set that allows multiple orders to be matched with one or more complementary orders. This would extend the exchange contract with batch functions for each matching strategy (see [ZEIP 40](https://github.com/0xProject/ZEIPs/issues/40)).

## Motivation

This enables more efficient order matching when orders are not maximally filled with a single call to `matchOrders` or `matchOrdersWithMaximalFill`. One use case is market fills on matching relayers, where a single taker order is matched against one or more maker orders:

1. A taker wants to fill one or more maker orders
2. They submit to the relayer a single complementary taker order
3. The relayer calls the exchange contract with the respective maker orders and complementary taker order

## Specification

### Interface

Below are the interfaces for the proposed batch matching functions.

```
/// @dev Match complementary orders that have a profitable spread.
///      Each order is filled at their respective price point, and
///      the matcher receives a profit denominated in the left maker asset.
/// @param leftOrders Set of orders with the same maker / taker asset.
/// @param rightOrders Set of orders to match against `leftOrders`
/// @param leftSignatures Proof that left orders were created by the left makers.
/// @param rightSignatures Proof that right orders were created by the right makers.
/// @return batchMatchedFillResults Amounts filled and profit generated.
function batchMatchOrders(
    LibOrder.Order[] memory leftOrders,
    LibOrder.Order[] memory rightOrders,
    bytes[] memory leftSignatures,
    bytes[] memory rightSignatures
)
public
nonReentrant
returns (LibFillResults.BatchMatchedFillResults memory batchMatchedFillResults);

/// @dev Match complementary orders that have a profitable spread.
///      Each order is maximally filled at their respective price point, and
///      the matcher receives a profit denominated in either the left maker asset,
///      right maker asset, or a combination of both.
/// @param leftOrders Set of orders with the same maker / taker asset.
/// @param rightOrders Set of orders to match against `leftOrders`
/// @param leftSignatures Proof that left orders were created by the left makers.
/// @param rightSignatures Proof that right orders were created by the right makers.
/// @return batchMatchedFillResults Amounts filled and profit generated.
function batchMatchOrdersWithMaximalFill(
    LibOrder.Order[] memory leftOrders,
    LibOrder.Order[] memory rightOrders,
    bytes[] memory leftSignatures,
    bytes[] memory rightSignatures
)
public
nonReentrant
returns (LibFillResults.BatchMatchedFillResults memory batchMatchedFillResults);
```

### Fill Results

The [LibFillResults](https://github.com/0xProject/0x-monorepo/blob/development/contracts/exchange-libs/contracts/src/LibFillResults.sol) contract would be extended to include a results struct for batch matching. One FillResult is created for each order that is filled. The total profit denominated in the left and right maker assets is also included.

```
contract LibFillResults {
    ...
    struct BatchMatchedFillResults {
        FillResults[] left;              // Fill results for left orders
        FillResults[] right;             // Fill results for right orders
        uint256 profitInLeftMakerAsset;  // Profit taken from left makers
        uint256 profitInRightMakerAsset; // Profit taken from right makers
    }
}
```

### Implementation

The pseudocode below describes the implementation of each batch fill function.

```
// sanity checks
require(leftOrders.length > 0)
require(leftOrders.length == leftSignatures.length)
require(rightOrders.length > 0)
require(rightOrders.length == rightSignatures.length)

// init
leftIdx = 0
rightIdx = 0
leftOrder = leftOrders[0]
rightOrder = rightOrders[0]
leftSignature = leftSignatures[0]
rightSignature = rightSignatures[0]

// batch match
batchMatchResults = {}
while True:
    // perform match
    matchResult = match(leftOrder, rightOrder, leftSignature, rightSignature)
    batchMatchResults.left.append(matchResult.left)
    batchMatchResults.right.append(matchResult.right)
    batchMatchResults.profitInLeftMakerAsset += matchResult.profitInLeftMakerAsset
    batchMatchResults.profitInRightMakerAsset += matchResult.profitInRightMakerAsset

    // iterate left
    if isFilled(leftOrder):
      leftIdx += 1
      if leftIdx == leftOrders.length:
        // all left orders have been matched
        break
      leftOrder = leftOrders[leftIdx]
      leftSignature = leftSignatures[leftIdx]

    // iterate right
    if isFilled(rightOrder):
      rightIdx += 1
      if rightIdx == rightOrders.length:
        // all right orders have been matched
        break
      rightOrder = rightOrders[rightIdx]
      rightSignature = rightSignatures[rightIdx]

return batchMatchResults
```

## Notes

* Redundant asset data can be optimized out by the [0x ABI Encoder](https://github.com/0xProject/0x-monorepo/tree/development/packages/utils/src/abi_encoder)

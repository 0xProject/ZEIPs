## Preamble

    ZEIP: 15
    Title: Do not double check signatures of previously filled/cancelled orders
    Author: 0x Core Team
    Type: Standard Track
    Category (*only required for Standard Track): Core
    Status: Final
    Created: 2017-12-15

## Simple Summary (2 Sentences)

Improve gas effeciency by only validating signatures once.

## Abstract

Currently, if you fill (or partially fill) an order that was already filled or cancelled, the Exchange contract will re-evaluate the signature of the order. This ZEIP would remove the need to verify the signature of an already partially filled or cancelled order.

## Motivation

Gas savings for orders that have previously been partially filled.

## Specification

An order that has already been partially filled or cancelled always implies that the order has been approved by the maker. Skipping signature verification would save at least 3000 gas with no additional downside.

```
    if (filled[orderHash] == 0 && cancelled[orderHash] == 0) {
        require(isValidSignature(...);
    }
```

## Rationale

## Test Cases

## Implementation

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

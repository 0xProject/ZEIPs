# ZEIP-42

## Summary

The stake-based liquidity incentives proposed in ZEIP 31 (https://github.com/0xProject/ZEIPs/issues/31) specify a protocol fee for each fill executed on the Exchange contract. This proposal defines the fees for each function in the Exchange.

Discussion: [ZEIP-42 Discussion](https://github.com/0xProject/ZEIPs/issues/42).

## Fees

### Exchange Fees

Each fill on the Exchange contract will incur a static fee that scales linearly with the gas price of the transaction. A multiplier, the protocolFeeMultiplier will be set in the Exchange contract. This multiplier will be multiplied by a transaction’s gas price to calculate the protocol fee that should be paid.

A protocol fee will be paid for each order that is “filled” (even if this is only a partial fill). As an example, a successful call to matchOrders will pay twice as many protocol fees as a successful call to fillOrder because two orders will be filled in the order matching operation. This behavior also applies to batch functions.

The initial proposed value for the protocolFeeMultiplier is 150k, roughly the cost of filling an ERC20 order through the exchange. While this value is not expected to change, it is upgradeable and can be updated after a time-lock has elapsed.

| Exchange Function               | Fee                                                  |
| ------------------------------- | ---------------------------------------------------- |
| batchCancelOrders               | N/A                                                  |
| batchFillOrKillOrders           | multiplier x <number of orders filled> x tx.gasPrice |
| batchFillOrders                 | multiplier x <number of orders filled> x tx.gasPrice |
| batchFillOrdersNoThrow          | multiplier x <number of orders filled> x tx.gasPrice |
| batchMatchOrders                | multiplier x <number of orders filled> x tx.gasPrice |
| batchMatchOrdersWithMaximalFill | multiplier x <number of orders filled> x tx.gasPrice |
| cancelOrder                     | N/A                                                  |
| cancelOrdersUpTo                | N/A                                                  |
| fillOrKillOrder                 | multiplier x tx.gasPrice                             |
| fillOrder                       | multiplier x tx.gasPrice                             |
| fillOrderNoThrow                | multiplier x tx.gasPrice                             |
| marketBuyOrders                 | multiplier x <number of orders filled> x tx.gasPrice |
| marketBuyOrdersNoThrow          | multiplier x <number of orders filled> x tx.gasPrice |
| marketSellOrders                | multiplier x <number of orders filled> x tx.gasPrice |
| marketSellOrdersNoThrow         | multiplier x <number of orders filled> x tx.gasPrice |
| matchOrders                     | multiplier x 2 x tx.gasPrice                         |
| matchOrdersWithMaximalFill      | multiplier x 2 x tx.gasPrice                         |

### Fee Assets

The only assets that will be used to pay protocol fees will be ether and ether that has been wrapped by the canonical weth contract (https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2#code). The inclusion of WETH as a protocol fee asset improves the user and developer experience of using protocol fees in many cases, especially when a contract is interacting with the protocol.

If enough ether is sent to the Exchange contract to pay the protocol fee, the ether will be used to pay the fee. In case there is not enough ether to pay the protocol fee, the staking contract can transfer WETH from the address that called the Exchange contract to pay the protocol fee. If WETH is taken to pay the protocol fee, any ether that was sent to the Exchange contract will be completely refunded.

To use WETH to pay protocol fees, one must first set an allowance for the Staking contract address. Otherwise, ether can be used to pay the fee.

### Meta-transactions

The execute and batchExecute functions are also required to pay protocol fees when calling any functions that require protocol fees. If enough ether is sent to the Exchange contract by the sender of the meta-transaction to pay the protocol fee, this ether will actually be used to pay the fee (and the sender will not be refunded by the creator of the meta-transaction). Otherwise, WETH will be taken from the signerAddress of the meta-transaction.

The decision to forego refunding the signerAddress was influenced by the fact that some senders of meta-transactions may intentionally subsidize the protocol fee that must be paid. This mechanism allows fees to be paid by either the sender or the signerAddress.

Charging the signerAddress WETH for protocol fees opened up an attack vector that would allow the sender of meta-transactions to grief the signerAddress out of WETH by submitting transactions with arbitrarily high gas prices. To mitigate this attack, a gasPrice field has been added to the ZeroExTransaction struct. execute and batchExecute will now verify that tx.gasPrice == transaction.gasPrice.

### Exception For Order Matching

In the original ZEIP, there was discussion around making an exception for one of the protocol fees in matchOrders for addresses that the exchange contract could identify as a matching relayer. After further discussion, we decided not to include this exception in an effort to reduce the complexity of V1 of the staking contracts.

## Notes

- A fee is only incurred by successful transactions
- In batch/market functions, each maker is credited with a portion of fee that is proportional to the number of their orders that were filled
- Any ETH sent to the Exchange contract in excess of the protocol fee is refunded to the sender
- If too little ETH is sent to the Exchange contract, WETH will be used to pay the protocol fee.

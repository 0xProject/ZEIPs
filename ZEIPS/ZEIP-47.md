## Preamble

    ZEIP: 47
    Title: ERC20BridgeProxy
    Author: 0x Core Team
    Type: Standard Track
    Category: Core
    Status: Final
    Created: 2019-07-01

Discussion: #47

## Simple Summary

A pluggable asset proxy for transferring ERC20 maker assets to a taker.

## Abstract

Introduce a new asset proxy that allows for a custom contract to be called that transfers ERC20 maker tokens to the taker. The asset proxy simply calls this contract and asserts that the taker's balance has increased by the required `amount`.

## Motivation

Liquidity is a primary concern for the 0x ecosystem. One potentially impactful way to supplement liquidity is to source liquidity from other on-chain exchanges and networks. By introducing a new asset proxy that is able to exchange taker tokens for maker tokens in an arbitrary manner, we can tap into these other liquidity sources.

## Specification

A custom "bridge" contract is specified in the asset data for this asset proxy. This bridge contract is called, converting ERC20 taker tokens into maker tokens. The asset proxy then confirms that the taker's balance has increased by the `amount` required.

### Asset data

```solidity
abi.encodeWithSelector(
    ERC20BridgeProxy(0).PROXY_ID,
    // Maker token address
    address tokenAddress
    // Address of the bridge contract
    address bridgeAddress,
    // Arbitrary data to pass to the bridge contract
    bytes bridgeData
);
```

The `bridgeContract` does the heavy lifting of actually swapping tokens and implements the `IERC20Bridge` interface:

### IERC20Bridge interface

```solidity
interface IERC20Bridge {
    bytes4 constant public BRIDGE_SUCCESS = 0xb5d40d78;

    function bridgeTransferFrom(
        bytes assetData,
        address tokenAddress,
        address from,
        address to,
        uint256 amount
    )
        external
        returns (bytes4 success);
}
```

## Flow

1. Order is created with `makerAddress` as the `IERC20Bridge` contract, with a `Wallet` signature type.
2. Taker calls `fillOrder()` on this order.
    1. taker tokens are transferred from taker to maker (`IERC20Bridge`).
    2. `ERC20BridgeProxy.transferFrom()` is invoked to transfer `amount` of maker tokens to the taker.
        1. `ERC20BridgeProxy` uses `IERC20(makerToken).balanceOf()` to get the maker token balance of the taker.
        2. `ERC20BridgeProxy` calls `bridgeContract.bridgeTransferFrom()`.
            1. `IERC20Bridge` interacts with on-chain liquidity sources to swap taker tokens for maker tokens and sending them to the taker.
        4. If `bridgeContract.bridgeTransferFrom()` does not return the magic value `0xb5d40d78`, revert.
        5. If the maker token balance of the taker has not increased by at least `amount`, revert.
    3. Taker now has maker tokens!

## Pros and Cons

### PROS

* It’s much easier to roll out support for more liquidity sources using this pattern.
* Anyone can create a buyer contract for use with this asset proxy.
* Buyer contracts have extreme flexibility with what they can do. We might see some really novel use cases.
* Neither the asset proxy nor buyer contracts are granted allowances in this pattern, so they can’t spend funds they don’t have.

## Implementation

The `ERC20BridgeProxy` would look like:

```solidity
contract ERC20BridgeProxy {
    bytes4 constant public BRIDGE_SUCCESS = 0xb5d40d78;

    function transferFrom(
        bytes assetData,
        address from,
        address to,
        uint256 amount
    )
        external
    {
        (
            address tokenAddress,
            address bridgeAddress,
            bytes bridgeData
        ) = abi.decode(
            assetData.sliceDestructive(4),
            (address, address, bytes)
        );
        uint256 balanceBefore = balanceOf(tokenAddress, to);
        bytes4 result = IERC20Bridge(bridgeAddress)
            .bridgeTransferFrom(bridgeData, tokenAddress, from, to, amount);
        require(result == BRIDGE_SUCCESS, "BRIDGE_FAILED");
        uint256 balanceAfter = balanceOf(tokenAddress, to);
        require(balanceBefore.safeAdd(amount) >= balanceAfter, "BRIDGE_UNDERPAY");
    }
}
```

A `bridgeContract` for Eth2Dai would look like:

```solidity
contract Eth2DaiBridge is
    ERC20Bridge
{
    address constant public ETH2DAI_ADDRESS = 0x39755357759ce0d7f32dc8dc45414cca409ae24e;
    address constant public WETH_ADDRESS = 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2;
    address constant public DAI_ADDRESS = 0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359;

    constructor() public {
        // Grant the Eth2Dai contract unlimited WETH and DAI allowance.
        IERC20Token(WETH_ADDRESS).approve(ETH2DAI_ADDRESS, uint256(-1));
        IERC20Token(DAI_ADDRESS).approve(ETH2DAI_ADDRESS, uint256(-1));
    }

    function bridgeTransferFrom(
        bytes bridgeData,
        address toTokenAddress,
        address from,
        address to,
        uint256 amount
    )
        external
        returns (bytes4 success)
    {
        // The "from" token is the opposite of the "to" token.
        IERC20Token fromToken = IERC20Token(WETH_ADDRESS);
        IERC20Token toToken = IERC20Token(DAI_ADDRESS);
        // Swap them if necessary.
        if (toTokenAddress == address(fromToken)) {
            (fromToken, toToken) = (toToken, fromToken);
        } else {
            require(
                toTokenAddress == address(toToken),
                "INVALID_ETH2DAI_TOKEN"
            );
        }
        // Try to sell all of this contract's `fromToken` balance.
        uint256 boughtAmount = IEth2Dai(ETH2DAI_ADDRESS).sellAllAmount(
            address(fromToken),
            fromToken.balanceOf(address(this)),
            address(toToken),
            amount
        );
        // Transfer the converted `toToken`s.
        toToken.transfer(to, boughtAmount);
        return BRIDGE_SUCCESS;
    }

    // `SignatureType.Wallet` callback, so that this bridge can be the maker
    // and sign for itself in orders. Always succeeds.
    function isValidSignature(
        bytes32 orderHash,
        bytes calldata signature
    )
        external
        view
        returns (bytes4 magicValue)
    {
        return LEGACY_WALLET_MAGIC_VALUE;
    }
}
```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

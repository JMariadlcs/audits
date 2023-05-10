# [L-01] Missing Zero Address check

## Lines of Code

## Vulnerability Details

### Description

Users can make deposits of ERC721, baseTokens, and nativeETH, but the protocol does not track how much each user has deposited.

### Impact

In the future, if a functionality such as an airdrop requires knowledge of how much each user has deposited on the protocol, it will not be possible to implement it.

### Tools Used

Manual review

### Recommended Mitigation Steps

Include one or more mappings to track the amount of ERC721, baseTokens, or nativeETH each user has deposited.

---

# [L-02] Change important variables without informing

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602

## Vulnerability Details

## Description

These functions are used to modify variables that play an important role in the protocol. Changing them directly, without alerting the users of the change, can lead to situations where users are unaware of the modification.

## Tools Used

Manual review.

## Recommended Mitigation Steps

Implement a mechanism that ensures a time period between the submission of the variable change by the owner and the actual modification of these variables.

# [L-03]

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## Vulnerability Details

### Description

All the functions inside PrivatePool are implemented for handling both, native ETH and other ERC20 as base tokens but in [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L44) it is said that `The only base token which is supported is native ETH`.

### Tools Used

Manual review.

### Recommended Mitigation Steps

Consider commenting why other ERC20 tokens are accepted as base tokens or change the implementation of PrivatePool.sol for only allow native ETH as base token.

# [L-04] Slippage not checked

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L204
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L293

## Vulnerability Details

### Description

It is said `DO NOT call this function directly unless you know what you are doing. Instead, use a wrapper contract that` but these function are from the EthRouter.sol contract [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L129) and [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L196) without checking the slipagge.
In the case of the sell function, the output amount is tried to be checked [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203-L205) but as anyone can send ETH to the contract using the `receive()` function, the slippage check is not correctly done.

### Impact

It can lead to a sell transaction with an unexpected amount of slippage for the user.

### Tools Used

Manual review.

### Recommended Mitigation Steps

Check slippage.

# [L-5] Unnecesary check statment

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225

## Vulnerability Details

### Description

According to [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L44) the only supported native token is native ETH, if the implementation is changed for actually only support native ETH, then the `baseToken != address(0)` is not necessary [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225) .

### Tools Used

Manual review

### Recommended Mitigation Steps

Change to `if (msg.value == 0) revert InvalidEthAmount();`

# [L-6] No reason for depositing

## Vulnerability Details

### Description

Users can deposit liquidity into a private pool but there is not benefit for doing it. The will earn nothing from providing liquidity to the private pool.

### Impact

As there is no benefit from depositing liquidity, any user wont do it so the pools will remain with a low liquidity balance.

### Tools Used

Manual review

### Recommended Mitigation Steps

Implement a mechanism for the users that have added liquidity to gain some earnings. For example earning fees from trades on the pool.

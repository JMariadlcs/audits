# [H-01] Users can withdraw more ETH than corresponding.

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141-L143
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290-L292

## Vulnerability Details

### Description

The intended behavior of these lines is to refund the user the surplus ETH from their transaction that was not used. However, since the EthRouter implements a [receive() function](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88), anyone can send ETH to the contract for any reason. Therefore, these lines may transfer not only the surplus ETH but the entire amount of ETH that is stored in the contract to the user

### Impact

If the contract holds any amount of ETH, users can perform buy, sell, and change operations and will receive the whole amount of ETH stored in the contract, rather than just the surplus ETH from the transaction.

### Tools Used

Manual review

### Recommended Mitigation Steps

Implement a mechanism to check the amount of ETH that has been spent on the transaction and compare it with the total amount of ETH sent in the transaction. Only transfer the remaining amount of ETH back to the caller. Not the whole contract balance.

---

# [H-02] Revert check can be bypassed

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203

## Vulnerability Details

### Description

Due to the [receive() function](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88) a user can send ETH to the contract or may the contract receive ETH for another reason, so the address(this).balance can be non-zero. If one execution returns that `minOutputAmount` is 0, will cause that the tx will not revert.

### Impact

Imagine that there is a balance of 1 ETH in the contract for whatever reason (e.g., someone sent it with receive). Now suppose a user wants to sell 1 NFT with a sell price of 0.1 ETH but sets minOutputAmount to 0.2 ETH. In this situation, the transaction should revert because the user theoretically will receive 0.1 ETH, which is less than 0.2 ETH. However, because there is 1 ETH on the contract, the transaction won't revert. Instead, the user will receive the total balance of the contract (1 ETH), and not the 0.2 ETH they expected.

### Tools Used

Manual review

### Recommended Mitigation Steps

Instead of comparing `minOutputAmount` with `address(this).balance` compare it with the return value `netOutputAmount` that the function `sell` from `PrivatePool.sol` returns and `outputAmount` that the function `nftSell` in `Pair.sol` returns.

---

# [H-03] Unsafe downcasting

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324

## Vulnerability Details

### Description

For all of the mentioned lines there is a downcasting from `uint256` to `uint128`

### Impact

The maximum value for a `uint128` is `(2^128) - 1`, this means that if the value that is trying to be casted is greater than `(2^128) -1` it will be wrongly trancated.

### Tools Used

Manual review

### Recommended Mitigation Steps

Use `uint256` instead of trying to downcast to `uint128`.

---

# [H-04] Virtual reserve variables can be wrongly set when a pool is created.

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L112
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L120
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L74
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L75

## Vulnerability Details

### Description

When a new pool is created, the variables `_virtualBaseTokenReserves` and `_virtualNftReserves` are manually set by the owner as parameters. These variables represent the number of base tokens and NFTs a pool contains. However, it is important to note that the owner can set arbitrary values to these variables, potentially resulting in a discrepancy between the actual and virtual reserve of base tokens and ERC721 NFTs held by the pool. Therefore, it is crucial for the owner to set the correct values to ensure that the pool functions as intended.
The real number of base tokens that are sent to the pool is defined as `baseTokenAmount` and the number of ERC721 as `tokenIds.length`. `baseTokenAmount` and `tokenIds` are also input parameters for the `create` function.

### Impact

The variables `_virtualBaseTokenReserves` and `_virtualNftReserves` play a crucial role in the protocol, as they are used in various functions such as `price`, `buy`, `sell`, `change`, `buyQuote`, and `sellQuote`. These variables determine the total amount of base tokens and ERC721 NFTs available in the pool and are used to calculate the required amount of tokens for buys, changes, and output amount for selling. However, if these variables are not set correctly, the actual liquidity of the base token and ERC721 tokens in the pool may not be accurately reflected, potentially leading to incorrect calculations for the required amount of tokens. Therefore, it is important for the owner to set these variables accurately to ensure the proper functioning of the pool.

Notice that also the owner can change these variables when he wants using the [setAllParametes](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L603) function or the individual ones.

### Tools Used

Manual review

### Recommended Mitigation Steps

Instead of manually input the values for `_virtualBaseTokenReserves` and `_virtualNftReserves` when creating the pool set them as:

- `_virtualBaseTokenReserves = baseTokenAmount` and `_virtualNftReserves = tokenIds.length`.
- Do not allow the owner to change the value of these variables once the pool in created. Even if the owner is trusted, he can set wrong values by accident.

---

# [H-05] Reentrancy Attack

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L240

## Vulnerability Details

### Description

The `buy` function in the PrivatePool.sol contract is used to buy NFTs from the pool, paying with base tokens from the caller.

When the function is called, the ERC721 tokens whose tokenIds are introduced as parameters are sent to the caller before the corresponding amount of base tokens for the buy are sent from the caller to the pool.

This means that if the caller is not an EOA but a smart contract that implements an onERC721Received function, once the ERC721 token is sent to the contract, the onERC721Received function will be triggered. Inside this function, there can be another call to the buy function of the PrivatePool.sol contract, allowing the attacker to withdraw all the ERC721 tokens inside the pool before any of the base tokens are transferred to the pool.

### Impact

This reentrancy attack allows the attacker to drain all the ERC721 tokens inside the pool.

### Tools Used

Manual review

### Recommended Mitigation Steps

First transfer the baseTokens from the caller to the pool and then sent the ERC721 to the caller, not the other way arround.

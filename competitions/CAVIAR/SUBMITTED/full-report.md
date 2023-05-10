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

---

# [M-01] No public function to withdraw

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484

## Vulnerability Details

### Description

There is a function available for users to deposit both ERC721 tokens and base tokens. However, there is no function available for users to withdraw tokens. Only the owner of the tokens has the ability to initiate withdrawals.

### Impact

Some users may assume that because there is a deposit function available, they will also be able to withdraw the liquidity they have deposited in the pool. However, since only the owner is authorized to initiate withdrawals, users will not be able to withdraw their tokens on their own. Moreover, if a user wants to retrieve their ERC721 and base tokens from the pool, but the owner refuses to return them, the user will not have any control over the situation. This situation lead to a loss of asset from the user point of view.

### Tools Used

Manual review

### Recommended Mitigation Steps

Add a public withdraw function that user can call to claim back the assets they have deposited on the private pool.

---

# [M-02] No check for input address

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135

## Vulnerability Details

### Description

There is no check for input addresses, so the code should be updated to ensure that the input address is a contract and not the null address (address(0)).

### Impact

If the input address is not a smart contract (meaning it has no code size) and is an externally owned account (EOA), or the null address (address(0)), the functionality of the private pool implementation and metadata will not perform as expected.

### Tools Used

Manual review

### Recommended Mitigation Steps

A mechanism for checking the input should be implemented. For example, create a internal function that checks the input address and call it on the mentioned lines.

```bash=
function checkAddress(address _address) internal view returns (bool) {
    uint256 codeSize;
    assembly { codeSize := extcodesize(_address) }
    return (codeSize > 0 && _address != address(0));
}

```

For checking the result add this require in the mentioned lines of the code.

```bash=
bool is_valid = checkAddress(_address);
require(is_valid, "Invalid address");
```

---

# [M-03] Wrong parameter can lead to DOS

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642

## Vulnerability Details

### Description

The `IERC3156FlashBorrower` interface implements the function `onFlashLoan` in the following way

```bash=
function onFlashLoan(
        address initiator,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) external returns (bytes32);
```

See reference [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/interfaces/IERC3156FlashBorrower.sol).

In [line 642](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642) the third param is `tokenId` instead of `amount`.

### Impact

The `flashLoan` is implemented for doing flashloans with only 1 token but in the case that the selected token has a tokenId different from 1 the function will not return success so the transaction will revert leading to a Denial of Service (DOS)

### Tools Used

Manual review

### Recommended Mitigation Steps

Change the third parameter of the `onFlashLoan` call to 1.

---

# [M-04] Down rounding can lead to user not receiving correct amount

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719

## Vulnerability Details

### Description

Not using mulDivUp can cause that the calculation of outputAmount can lead to a non integer result that will be rounded down.

Assume that:

- inputAmount = 1e18
- virtualBaseTokenReserves = 1e7
- virtualNftReserves = 5e18

The result of ` uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);` will be 9999999.9999 that will be rounded to 9999998

### Impact

If the calculation of the outputAmount is rounded down, the user will receive fewer tokens than the actual amount that is owed to them.

In the example shown above the loss impact is not very high but it exists.

### Tools Used

Manual review

### Recommended Mitigation Steps

Rounding up the required amount is advisable, and it can be achieved by utilizing solmate's `FixedPointMathLib` library to compute the quote and round it up. By doing so, the required amount will always be a minimum of 1 wei.

Change line 719 by:

```bash=
uint256 outputAmount = mulDivUp(inputAmount, virtualBaseTokenReserves, (virtualNftReserves + inputAmount));
```

---

# [M-05] Down rounding can lead to revert tx

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182

## Vulnerability Details

### Description

If the calculation of `salePrice` as `uint256 salePrice = inputAmount / buys[i].tokenIds.length;` gives a result less than 1 for any reason, for example 0.9, it will be rounded to 0.

### Impact

If `salePrice` is rounded to 0, `royaltyFee` that is calculated as `(salePrice * royalty.royaltyFraction) / _feeDenominator();` will be also 0 and the royalty receiver will receive any royalty fee.

### Tools Used

Manual review

### Recommended Mitigation Steps

Rounding up the required amount is advisable, and it can be achieved by utilizing solmate's `FixedPointMathLib` library to compute the quote and round it up. By doing so, the required amount will always be a minimum of 1 wei.

Change [line 115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115) to:

```bash=
uint256 salePrice = mulDivUp(inputAmount, 1, buys[i].tokenIds.length);
```

Change [line 182](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182) to:

```bash=
uint256 salePrice = mulDivUp(outputAmount, 1, sells[i].tokenIds.length);
```

---

# [M-06] Tokens with more than 18 decimals will cause tx revert on deposit function (DOS)

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L233

## Vulnerability Details

### Description

The function .price() may return a price with less than 18 decimals.

The [comment](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L743) states that it ensures the exponent is 18 decimals, but if the baseToken has more than 18 decimals, for example, 19 decimals, `exponent` will be calculated as `36-19` so will be equal to `18`.

Assumming that `virtualBaseTokenReserves` and `virtualNftReserves` has the same decimal precission, then the function `price()` will return a value with `17` decimal precision, 1e17, which is less than 18 and different from the baseToken's decimal precission.

### Impact

Taking into consideration what is being mentioned above, if a user inputs maxPrice and minPrice in the [deposit function](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L223-L224) expressed in a magnitude different from the one returned from the price, the comparison may not be done correctly. For instance, if the user sets minPrice = 1e18 and .price() returns 1e17 due to the base token having 19 decimals, the transaction will revert.

It is a common situation for users to input maxPrice and minPrice with the same decimal precision as the base token. However, if the base token has more than 18 decimals, as described above, the transaction will revert. This could lead to a denial of service when using the deposit function in a private pool with this base token.

### Tools Used

Manual review

### Recommended Mitigation Steps

The ideal solution is to ensure that the comparison between the value returned from the .price() function and maxPrice and minPrice is done with the same decimal precision. Assuming that users will use the same decimal precision as the base token for the maxPrice and minPrice variables, the .price() function should return the price directly following the baseToken's decimal precision.

To achive this goal, change this [line](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744) to:

```bash=
uint256 exponent = baseToken == address(0) ? 18 : (ERC20(baseToken).decimals());
```

---

# [M-07] Possible underflow cause tx reverted leading to DOS

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L723

## Vulnerability Details

### Description

The function `sellQuote` returns `netOutputAmount` which is calculated as `netOutputAmount = outputAmount - feeAmount - protocolFeeAmount;`

The code for calculating the other variables is [this one](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719-L723):

```bash=
 uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);

protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;
feeAmount = outputAmount * feeRate / 10_000;
netOutputAmount = outputAmount - feeAmount - protocolFeeAmount;
```

The maximum value for `feeRate` is `5000` which correspond to a 50% fee.
Assume that `feeRate` was set to `4000`, which is a 40% fee.

`protocolFeeRate` has not maximum value so assume that it was set to `7000` which corresponds to 70% fee.

So `protocolFeeAmount` will be `outputAmount * 7000 / 10_000`.
`feeAmount` will be `outputAmount * 5000 / 10_000`.

So `netOutputAmount` will be:

`netOutputAmount = outputAmount - (outputAmount * 5000 / 10_000) - (outputAmount * 7000 / 10_000);`

which lead to:
`netOutputAmount = outputAmount - (outputAmount * 12_000 / 10_000);`

Taking into account that `outputAmount` is calculated as `inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);`

In a case where `virtualBaseTokenReserves` and `virtualNftReserves` are both set to `1e18`, for example, and `inputAmount` is also 1e18, `outputAmount` will be `5e17`.

Going back to `netOutputAmount = outputAmount - (outputAmount * 12_000 / 10_000);`. With the calculated value for outputAmount of `5e17`, netOutputAmount will underflow and revert.

### Impact

Following the above explained example, another combination for `feeRate` and `protocolFeeAmount` can lead to the same situation of underflow that will make that the user is not able to perform the sell transaction.

Therefore this lead to denial of service (DOS) of the sell function.

### Tools Used

Manual review

### Recommended Mitigation Steps

Establish a maximum value for `protocolFeeRate` of `5000` (50%), this will make that `netOutputAmount = outputAmount - (outputAmount * 10_000 / 10_000);` is the maximum case that can be reached and this does not revert.

---

# [M-08] Division by 0 value

## Lines of Code

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L701

## Vulnerability Details

### Description

The denominator of the operation is `virtualNftReserves - outputAmount`.

- `outputAmount` is the amount of NFTs to buy multiplied by 1e18.
- As the variable `_virtualNftReseves` is set by the pool creator, it can be set to a specific value so when performing this operation, the substraction result value is 0 and thus the denominator is 0 so the tx will revert.

### Impact

Imagin a situation where `outputAmount` is 1e18 because the user wants to buy 1 NFT.
Assume that in this example, `_virtualNftReseves` is also 1e18.

The division will not be able to be performed as the denominator is 0.
The method `mulDivUp` implements a [require](https://github.com/transmissions11/solmate/blob/main/src/utils/FixedPointMathLib.sol) for checking if the denominator is 0, if it is, it reverts.

### Tools Used

Manual review

### Recommended Mitigation Steps

Reimplement the `buyQuote` function to calculate a inputAmount in a way that do not allow the denominator to be 0 in any case.

---

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

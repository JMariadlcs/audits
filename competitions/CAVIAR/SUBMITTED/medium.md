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

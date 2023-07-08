# Swap leftovers ETH are locked in the JBXBuybackDelegate

# Lines of code

[https://github.com/R-E-A-C-H/blackpanda-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L263-L275](https://github.com/R-E-A-C-H/blackpanda-juicebox/blob/9a36e5c8d0588f0f262a0cd1c08e34b2184d8f4d/juice-buyback/contracts/JBXBuybackDelegate.sol#L263-L275)

# Vulnerability details

## Vulnerability details

In case that the project JBToken address is bigger than WETH address, `_projectTokenIsZero` is set to false. The test cases of buyback delegate only cover the situation, where the JBToken is lower than WETH.

```
    constructor(
        IERC20 _projectToken,
        IWETH9 _weth,
        IUniswapV3Pool _pool,
        IJBPayoutRedemptionPaymentTerminal3_1 _jbxTerminal
    ) {
        projectToken = _projectToken;
        pool = _pool;
        jbxTerminal = _jbxTerminal;
        _projectTokenIsZero = address(_projectToken) < address(_weth); // <======= sets tokens ordering for UniswapV3
```

In uniswap v3 pool swapping, whenever user provides more ETH than what ProjectToken is worth of, then the remaining ETH will be send back to swap() caller, in this case, JBXBuybackDelegate.  
Since this ETH are not sent back to original caller i.e., payer, its stuck inside JBXBuybackDelegate contract.

## Impact

Users are loosing Uniswap swap leftovers

## Proof of Concept

Unfortunately this test is hard to setup with test cases provided by JuiceBox team, due to specific requirements concerning project setup and block height. We came up with the test, to verify the case in which the non-WETH token is considered as project token and is bigger than WETH. Please create a new file in test folder and run `forge test` to test it:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
interface IPool {
    function swap(
        address recipient,
        bool zeroForOne,
        int256 amountSpecified,
        uint160 sqrtPriceLimitX96,
        bytes calldata data
    ) external returns (int256 amount0, int256 amount1);
    function token0() external view returns(address);
    function token1() external view returns(address);
}

interface IERC20 {
    function balanceOf(address) external view returns (uint256);
}

interface IWETH9 {
    function deposit() external payable;
    function transfer(address, uint256) external;
    function balanceOf(address) external view returns (uint256);
}

contract Swapper {
    IPool pool;

    IERC20 projectToken;
    IWETH9 weth = IWETH9(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2);

    bool _projectTokenIsZero;

    uint160 internal constant MIN_SQRT_RATIO = 4295128739;
    uint160 internal constant MAX_SQRT_RATIO = 1461446703485210103287273052203988822378723970342;

    constructor(IPool p) {
        pool = p;
        address pt = pool.token0();
        if (pool.token0() == address(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2)) {
            pt = pool.token1();
        }

        projectToken = IERC20(pt);

        _projectTokenIsZero = address(projectToken) < address(weth);
    }

    function check_swap(uint amountIn, uint minOutput) external payable returns(int256, int256) {
        (int256 amount0, int256 amount1) = pool.swap({
            recipient: address(this),
            zeroForOne: !_projectTokenIsZero,
            amountSpecified: int256(amountIn),
            sqrtPriceLimitX96: _projectTokenIsZero ? MAX_SQRT_RATIO - 1 : MIN_SQRT_RATIO + 1,
            data: abi.encode(minOutput)
        });

        return (amount0, amount1);
    }

    function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata data) external {
        // Check if this is really a callback
        require(msg.sender == address(pool), "Not Authorized");

        // Unpack the data
        (uint256 _minimumAmountReceived) = abi.decode(data, (uint256));

        // Assign 0 and 1 accordingly
        uint256 _amountReceived = uint256(-(_projectTokenIsZero ? amount0Delta : amount1Delta));
        uint256 _amountToSend = uint256(_projectTokenIsZero ? amount1Delta : amount0Delta);

        // Revert if slippage is too high
        require(_amountReceived >= _minimumAmountReceived, "min amount");

        // Wrap and transfer the weth to the pool
        weth.deposit{value: _amountToSend}();
        weth.transfer(address(pool), _amountToSend);
    }
    receive() external payable {}
}
contract UniswapV3ETHRefundExploitTest is Test {
    // IPool pool = IPool(0x88e6A0c2dDD26FEEb64F039a2c41296FcB3f5640);
    IPool pool = IPool(0x7379e81228514a1D2a6Cf7559203998E20598346);

    IERC20 projectToken;
    IWETH9 weth = IWETH9(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2);

    function setUp() public {
        vm.createSelectFork("https://rpc.ankr.com/eth");

        vm.label(address(projectToken), "ProjectToken");
        vm.label(address(weth), "WETH");

        address pt = pool.token0();
        if (pool.token0() == address(0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2)) {
            pt = pool.token1();
        }

        projectToken = IERC20(pt);
    }

    function testExploit() public {
        console.log("Testing for a pool :", address(pool));
        console.log("================================================================");
        Swapper swapper = new Swapper(pool);

        vm.label(address(swapper), "swapper");

        address user = makeAddr("user");
        uint256 amountIn = 1000_000 ether;
        uint256 minOutput = 0;

        vm.label(user, "user");
        vm.deal(user, amountIn);

        emit log_named_decimal_uint("Before: Balance of Pool's WETH:", weth.balanceOf(address(pool)), 18);
        emit log_named_decimal_uint("Before: Balance of Pool's Project Token:", projectToken.balanceOf(address(pool)), 18);
        console2.log();

        emit log_named_decimal_uint("Before: Balance of ETH (swapper):", address(swapper).balance, 18);
        emit log_named_decimal_uint("Before: Balance of WETH (swapper):", weth.balanceOf(address(swapper)), 18);
        emit log_named_decimal_uint("Before: Balance of Project Token (swapper):", projectToken.balanceOf(address(swapper)), 18);
        console2.log();

        emit log_named_decimal_uint("Before: Balance of ETH (user):", user.balance, 18);
        emit log_named_decimal_uint("Before: Balance of WETH (user):", weth.balanceOf(user), 18);
        emit log_named_decimal_uint("Before: Balance of Project Token (user):", projectToken.balanceOf(user), 18);
        console2.log();

        vm.prank(user);
        (int256 amount0, int256 amount1) = swapper.check_swap{value: amountIn}(amountIn, minOutput);

        console2.log("After swap: Amount 0:", amount0);
        console2.log("After swap: Amount 1:", amount1);
        console2.log();

        emit log_named_decimal_uint("After: Balance of ETH (swapper):", address(swapper).balance, 18);
        emit log_named_decimal_uint("After: Balance of WETH (swapper):", weth.balanceOf(address(swapper)), 18);
        emit log_named_decimal_uint("After: Balance of Project Token (swapper):", projectToken.balanceOf(address(swapper)), 18);
        console2.log();

        emit log_named_decimal_uint("After: Balance of ETH (user):", user.balance, 18);
        emit log_named_decimal_uint("After: Balance of WETH (user):", weth.balanceOf(user), 18);
        emit log_named_decimal_uint("After: Balance of Project Token (user):", projectToken.balanceOf(user), 18);
        console2.log();

        emit log_named_decimal_uint("After: Balance of Pool's WETH:", weth.balanceOf(address(pool)), 18);
        emit log_named_decimal_uint("After: Balance of Pool's Project Token:", projectToken.balanceOf(address(pool)), 18);
    }
}
```

## Console Output

For **High** Liquidity Pool: 0x88e6A0c2dDD26FEEb64F039a2c41296FcB3f5640

```console
  Testing for a pool : 0x88e6A0c2dDD26FEEb64F039a2c41296FcB3f5640
  ================================================================
  Before: Balance of Pool's WETH:: 66269.602156132605170356
  Before: Balance of Pool's Project Token:: 0.000116014157471156

  Before: Balance of ETH (swapper):: 0.000000000000000000
  Before: Balance of WETH (swapper):: 0.000000000000000000
  Before: Balance of Project Token (swapper):: 0.000000000000000000

  Before: Balance of ETH (user):: 1000000.000000000000000000
  Before: Balance of WETH (user):: 0.000000000000000000
  Before: Balance of Project Token (user):: 0.000000000000000000

  After swap: Amount 0: -114230315452600
  After swap: Amount 1: 1000000000000000000000000

  After: Balance of ETH (swapper):: 0.000000000000000000
  After: Balance of WETH (swapper):: 0.000000000000000000
  After: Balance of Project Token (swapper):: 0.000114230315452600

  After: Balance of ETH (user):: 0.000000000000000000
  After: Balance of WETH (user):: 0.000000000000000000
  After: Balance of Project Token (user):: 0.000000000000000000

  After: Balance of Pool's WETH:: 1066269.602156132605170356
  After: Balance of Pool's Project Token:: 0.000001783842018556
```

For **Low** Liquidity Pool : 0x7379e81228514a1D2a6Cf7559203998E20598346

```console
  Testing for a pool : 0x7379e81228514a1D2a6Cf7559203998E20598346
  ================================================================
  Before: Balance of Pool's WETH:: 13972.824366210394385747
  Before: Balance of Pool's Project Token:: 29678.153610456388197439

  Before: Balance of ETH (swapper):: 0.000000000000000000
  Before: Balance of WETH (swapper):: 0.000000000000000000
  Before: Balance of Project Token (swapper):: 0.000000000000000000

  Before: Balance of ETH (user):: 1000000.000000000000000000
  Before: Balance of WETH (user):: 0.000000000000000000
  Before: Balance of Project Token (user):: 0.000000000000000000

  After swap: Amount 0: 29774384470067394065394
  After swap: Amount 1: -29670151004437125603226

  After: Balance of ETH (swapper):: 970225.615529932605934606
  After: Balance of WETH (swapper):: 0.000000000000000000
  After: Balance of Project Token (swapper):: 29670.151004437125603226

  After: Balance of ETH (user):: 0.000000000000000000
  After: Balance of WETH (user):: 0.000000000000000000
  After: Balance of Project Token (user):: 0.000000000000000000

  After: Balance of Pool's WETH:: 43747.208836277788451141
  After: Balance of Pool's Project Token:: 8.002606019262594213
```

## Tools Used

Manual analysis

## Recommended Mitigation Steps

We recommend either returning the funds to the caller, or minting the leftover amount to them.

## Assessed type

Uniswap

# What I Learned

- When uniswap v3 swap executes using `swap()`, we don't need to send ETH/Tokens before performing swap, as this is the case with V2 swap.
- When we swap using some v3 pool, in the callback it will send info about number of tokens it has sent and number of other tokens swapper need to send in the form of amount0 and amount1 parameter in callback.
- When we swap using HIGH liquidity pool, no matter whatever amount of ETH/Tokens we have, if liquidity of pool is enough to satisfy then in callback it will ask for a required amount of ETH/Tokens that need to send to pool and then it will complete swap.
- When we swap using LOW liquidity pool and if we send more liquidity than what is available in pool for swapping, then pool will perform swap with whatever liquidity is there and then it should be an responsibility of swap caller contract to return leftover funds to the actual caller.

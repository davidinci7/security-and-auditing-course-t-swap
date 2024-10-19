## High

### [H-1] Incorrect fee calculation in `TSwapPool::getInputAmountBasedOnOutput` causes protocol to take too many tokens from users, resulting in lost fees.

**Description** The `getInputAmountBasedOnOutput` function is intended to calculate the amount of tokens a user should deposit given an amount of output tokens. However, the function currently miscalculates the resulting amount. When calculating the fee, it scales the amount by 10_000 instead of 1_000.

**Impact** Protocol takes more fees than expected from users.

**Proof of Concepts** Include the following code in the `TswapPool.t.sol` file.

**Recommended mitigation** Consider making the following change to the function.

``` diff
    function getInputAmountBasedOnOutput(
        uint256 outputAmount,
        uint256 inputReserves,
        uint256 outputReserves
    )
        public
        pure
        revertIfZero(outputAmount)
        revertIfZero(outputReserves)
        returns (uint256 inputAmount)
    {
-       return ((inputReserves * outputAmount) * 10_000) / ((outputReserves - outputAmount) * 997);
+       return ((inputReserves * outputAmount) * 1_000) / ((outputReserves - outputAmount) * 997);
    }
```


## Medium

### [M-1] `TSwapPool::deposit` is missing deadline check causing transactions to complete even after the deadline

**Description** The `deposit` function accepts a deadline parameter, which according to the documentation is "The deadline for the transaction to be completed by".
However, this parameter is never used. As a consequence, operations that add liquidity to the pool might be executed at unexpected times, in market conditions where the deposit rate is unfavorable.

**Impact** Transactions could be sent when market condiions are unfavorable to deposit, even when adding a deadline parameter.

**Proof of Concepts** The `deadline` parameter is unused.

**Recommended mitigation** Consider making the following change to the function.

```diff
function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
+       revertIfDeadlinePassed(deadline)
        revertIfZero(wethToDeposit)
        returns (uint256 liquidityTokensToMint)
    {...}
```

## Low

### [L-1] `TSwapPool::LiquidityAdded` event has parameters out of order

**Description** When the `LiquidityAdded` event is emitted in the `TSwapPool::_addLiquidityMintAndTransfer` function, it logs values in an incorrect order. The `poolTokensToDeposit`
value should go in the third parameter position, whereas the `wethToDeposit` value should go second.

**Impact** Event emission is incorrect, leading to off-chain functions potentially malfunctioning.

**Recommended mitigation**

```diff
- emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
+ emit LiquidityAdded(msg.sender, wethToDeposit, poolTokensToDeposit);
```

## Informationals

### [I-1] `PoolFactory::PoolFactory__PoolDoesNotExist` is not used and should be removed

```diff
- error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

### [I-2] `PoolFactory::constructor` Lacking zero address checks

```diff
    constructor(address wethToken) {
+       if(wethToken == address(0)){
+           revert();
+       }
        i_wethToken = wethToken;
    }
```

### [I-3] `PoolFactory::createPool` should use `.symbol()` instead of `.name()`

```diff
-   string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
+   string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).symbol());
```

### [I-4] `TSwapPool::constructor` Lacking zero address check - wethToken & poolToken

```diff
    constructor(
        address poolToken,
        address wethToken,
        string memory liquidityTokenName,
        string memory liquidityTokenSymbol
    ) ERC20(liquidityTokenName, liquidityTokenSymbol) {
+       if(poolToken || wethToken == address(0)){
+          revert();
+        }
        i_wethToken = IERC20(wethToken);
        i_poolToken = IERC20(poolToken);
    }
```

### [I-5] `TSwapPool` events should be indexed

```diff
- event Swap(address indexed swapper, IERC20 tokenIn, uint256 amountTokenIn, IERC20 tokenOut, uint256 amountTokenOut);
+ event Swap(address indexed swapper, IERC20 indexed tokenIn, uint256 amountTokenIn, IERC20 indexed tokenOut, uint256 amountTokenOut);
```

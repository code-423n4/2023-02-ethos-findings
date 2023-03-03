# [G-01]  ++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)
##description 
++i costs less gas compared to i++ or i += 1 (same for --i vs i-- or i -= 1).
it Saves 5 gas per loop
## Codes 
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L108 
```   for(uint256 i = 0; i < numCollaterals; i++) { ```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L608
```for (vars.i = 0; vars.i < _n && vars.user != firstUser; vars.i++) {```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L690
```for (vars.i = 0; vars.i < _n; vars.i++) ```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808
```for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

```for (vars.i = 0; vars.i < _troveArray.length; vars.i++) {```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L56

```for(uint256 i = 0; i < _collaterals.length; i++) {```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L191

```i++``

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L113
``` for (uint256 i = 0; i < numCollaterals; i++) {```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/RedemptionHelper.sol#L154
```for (uint256 i = 0; i < numCollaterals; i++) {```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L120

```while (vars.currentTroveuser != address(0) && vars.remainingLUSD > 0 && _maxIterations-- > 0)```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L273
```        if (x % y != 0) q++;```

# [G‑02] Structs can be packed into fewer storage slots.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#58 

## Description: 

The struct ```ChainlinkResponse``` can be packed into fewer storage slots. 
```
struct ChainlinkResponse {
        uint80 roundId;
        int256 answer;
        uint256 timestamp;
        bool success;
        uint8 decimals;
    }
```

## Recommendation:
```
struct ChainlinkResponse {
        uint80 roundId;
        uint8 decimals;
        uint256 timestamp;
        int256 answer;
        bool success;
    }
```
# [G-03] <array>.length should not be looked up in every loop of a for-loop
Reading array length at each iteration of the loop consumes more gas than necessary.
In the best case scenario (length read on a memory variable), caching the array length in the stack saves around 3 gas per iteration. In the worst case scenario (external calls at each iteration), the amount of gas wasted can be massive.
Here, Consider storing the array’s length in a variable before the for-loop, and use this new variable instead.

Excluding the first loop costs, you also incur gas as follows:
-   storage arrays incur (100 gas)
-   memory arrays use MLOAD (3 gas)
-   calldata arrays use CALLDATALOAD (3 gas)

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L57

``` _collaterals.length ```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L808
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L882

```_troveArray.length;```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L810
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L831

``` collaterals.length```
# [G-04] Use shorter require() strings
## Description 
When these strings get longer than 32 bytes, they cost more gas.

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L323-326

```
require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == defaultPoolAddress,
            "ActivePool: Caller is neither BO nor Default Pool");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L331-336

```
require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress ||
            msg.sender == redemptionHelper ||
            msg.sender == stabilityPoolAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager nor StabilityPool");
```
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L340-343
```
require(
            msg.sender == borrowerOperationsAddress ||
            msg.sender == troveManagerAddress,
            "ActivePool: Caller is neither BorrowerOperations nor TroveManager");
```
# [G-05] Using 10**X for constants isn't gas efficient
## Description  
In Solidity, a constant expression in a variable will compute the expression everytime the variable is called. It's not the result of the expression that is stored, but the expression itself.

As Solidity supports the scientific notation, constants of form 10**X can be rewritten as 1eX to save the gas cost from the calculation with the exponentiation operator **.
## codes
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L40
```uint256 public constant DEGRADATION_COEFFICIENT = 10**18; ```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125
```        lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6;```

# [G-06] Use bytes32 rather than string.
## Description 
bytes32 uses less gas because it fits in a single word of the EVM, and string is a dynamically sized-type which has current limitations in Solidity. 
## codes
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30
``` string constant public NAME = "ActivePool";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L21
``` string constant public NAME = "BorrowerOperations";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L17
``` string constant public NAME = "CollSurplusPool"; ```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/DefaultPool.sol#L17
``` string constant public NAME = "DefaultPool";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L17
```   string constant public NAME = "HintHelpers";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L32-34

    string constant internal _NAME = "LUSD Stablecoin";
    string constant internal _SYMBOL = "LUSD";
    string constant internal _VERSION = "1";

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/PriceFeed.sol#L27
```string constant public NAME = "PriceFeed";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L49

```string constant public NAME = "SortedTroves";```

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150
``` string constant public NAME = "StabilityPool";```
# [G-07]  Use indexed events as they are less costly compared to non-indexed ones
##description
Using the indexed keyword for value types such as uint, bool, and address saves gas costs. However, this is only the case for value types, whereas indexing bytes and strings are more expensive than their unindexed version. 
## codes    
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58-L67
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L92-L102
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollSurplusPool.sol#L31-L34
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/HintHelpers.sol#L21-L23
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L78-L82
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/SortedTroves.sol#L51-L54

| | issue |
| ----------- | ----------- |
| 1 | [state variables can be packed into fewer storage slots](#) |
| 2 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#) |
| 3 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#) |
| 4 | [state variables should be cached in stack variables rather than re-reading them from storage](#) |
| 5 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#) |
| 6 | [not using the named return variables when a function returns, wastes deployment gas](#) |
| 7 | [can make the variable outside the loop to save gas](#) |
| 8 | [splitting require() statements that use `&&` saves gas](#) |
| 9 | [require() or revert() statements that check input arguments should be at the top of the function](#) |
| 10 | [Avoid a SLOAD optimistically](#) |
| 11 | [use a more recent version of solidity](#) |
| 12 | [internal functions only called once can be inlined to save gas](#) |
| 13 | [avoid an unnecessary sstore by not writing a default value for bools](#) |
| 14 | [Using fixed bytes is cheaper than using string](#) |
| 15 | [ Ternary over if ... else](#) |
| 16 | [public functions not called by the contract should be declared external instead](#) |
| 17 | [state var is defined but not used anywhere but](#) |
| 18 | [Optimize names to save gas](#) |
| 19 | [Non-strict inequalities are cheaper than strict ones](#) |
| 20 | [instead of assigning to zero use `delete`](#) |
| 21 | [precalculating part of code will reduce the number of operations being done](#) |
| 22 | [use argument instead of state var](#) |


## 1. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

`stabilityPoolAddress` and `initialized` can be put together in one slot instead of 2 if they are defined near each other. make `stabilityPoolAddress` right after L21 
- [CommunityIssuance.sol#L21-L39](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21-L39)

`mintingPaused` and `guardianAddress` can be put together to use one less slot. make `guardianAddress` right after L37
- [LUSDToken.sol#L37-L71](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37-L71)

`treasury` and `emergencyShutdown` can be put together in one slot. declare them near each other
- [ReaperVaultV2.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L49)

`emergencyExit` and `want` are in the same slot but they are never used together in a function. instead put `emergencyExit` and `vault` together which `emergencyExit` is always used with in the same functions. this will not save storage slot but will have cheaper reads to save gas.
- [ReaperBaseStrategyv4.sol#L58](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L58)


## 2. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

- [ReaperVaultV2.sol#L73](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73)
- [ReaperVaultV2.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L74)
- [ReaperVaultV2.sol#L75](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L75)
- [ReaperVaultV2.sol#L76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L76)

- [ReaperBaseStrategyv4.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49)
- [ReaperBaseStrategyv4.sol#L50](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L50)
- [ReaperBaseStrategyv4.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L51)
- [ReaperBaseStrategyv4.sol#L52](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L52)


## 3. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`troveManagers` and `stabilityPools` and `borrowerOperations` can be combined and should use a mapping of a address to a struct. they can be combined into one slot instead of 3. they are accessed in the same places so there will be extra gas saves too.
- [LUSDToken.sol#L62-L64](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L62-L64)


## 4. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by c4udit)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

use `_collateralConfigAddress` argument instead of `collateralConfigAddress` SLoad to save 100 gas.
- [ActivePool.sol#L105](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L105)

cache `defaultPoolAddress` before the if statement. this will risk wasting 3 gas but will save 297 gas if the condition happens
- [ActivePool.sol#L179-L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L179-L181)

cache `collSurplusPoolAddress` before the if statement. this will risk wasting 3 gas but will save 297 gas if the condition happens
- [ActivePool.sol#L182-184](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L182-184)

cache `troveManagerAddress` before L328. using cached version saves 97 gas here
- [ActivePool.sol#L328](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L328)

`currentEpoch` and `currentScale` should be cached before L434. they are used 4 times and they dont change in this part of code, so cache them and save 300 gas for each of them
- [StabilityPool.sol#L434-L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L434-L436)

cache `lastDistributionTime` before the if statement. we will risk using 3 extra gas but has a high possibility to save 100 or 200 gas depending on the ternary operation result
- [CommunityIssuance.sol#L86-L88](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86-L88)
- [CommunityIssuance.sol#L106-L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L106-L107)

cache `lastIssuanceTimestamp` before the if statement. we will risk using 3 extra gas but has a high possibility to save 100
- [CommunityIssuance.sol#L86-L90](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L86-L90)
- [CommunityIssuance.sol#L106-L107](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L106-L107)

cache `amount.div(distributionPeriod)` in some variable and give the cached version to `rewardPerSecond` in the next line so we can use the cached version in the emit of L116. this will save 97 gas
- [CommunityIssuance.sol#L112](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L112)

cache `stakes[_user]` before the for loop because it doesnt change with the value of `i` so it doesnt need to be read every loop. skips a complex storage read for every iteration.
- [LQTYStaking.sol#L209](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L209)

cache `F_Collateral[collateral]` in a uint variable and give `snapshots[_user].F_Collateral_Snapshot[collateral]` and `amounts[i]` the cached variable. this stops one complex storage read per iteration
- [LQTYStaking.sol#L230-L231](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L230-L231)

cache `troveManagerAddress` before L252. using cached version saves 97 gas here
- [ActivePool.sol#L252](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L328)

cache `strategies[_strategy].allocBPS` before L210. because its in the condition of if statement it will be checked once and if the check fails it will be read again in the next line. so we can cache it before the if statement and risk losing 3 gas to use one less complex SLOAD with a high chance of happening.
- [ReaperVaultV2.sol#L210](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L210)

cache `lockedProfit` inside the if statement. saves 100 gas
- [ReaperVaultV2.sol#L421](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L421)

cache `tvlCap` before the first if statement. there is a chance to save 200 gas with only risking losing 3 gas. mostly the checks fail and we save gas.
- [ReaperVaultERC4626.sol#L80-L82](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L80-L82)
- [ReaperVaultERC4626.sol#L123-L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L80-L82)

cache `vault` at the start of the function. saves 100 gas
- [ReaperBaseStrategyv4.sol#L111](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L111)

cache `want` at the start of the if statement. saves 100 gas
- [ReaperStrategyGranarySupplyOnly.sol#L179](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L179)


can lose 1 complex storage load per instance here. cache `collAmount[_collateral].sub(_amount)` in some stack variable and give the cached version to `collAmount[_collateral]` in the next line instead. with this change we can use the cached variable that we made instead of doing a complex read from storage. 
there is different codes but `same concept` is used so there is just one of them to show the concept
```
        collAmount[_collateral] = collAmount[_collateral].sub(_amount);
        emit ActivePoolCollateralBalanceUpdated(_collateral, collAmount[_collateral]);
```
change to 
```
        uint256 someVariable = collAmount[_collateral].sub(_amount);
        collAmount[_collateral] = someVariable
        emit ActivePoolCollateralBalanceUpdated(_collateral, someVariable);
```
- [ActivePool.sol#L175](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L175)
- [ActivePool.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L193)
- [ActivePool.sol#L200](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L200)
- [ActivePool.sol#L207](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L207)
- [StabilityPool.sol#L434-L436](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L434-L436)
- [LQTYStaking.sol#L183](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L183)

caching these next ones are the same concept as the last ones but they are only simple SLoads and only save 100 gas each
- [LQTYStaking.sol#L124](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L124)
- [LQTYStaking.sol#L159](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L159)
- [LQTYStaking.sol#L193](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L193)


## 5. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

- [ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
- [ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)
- [ReaperVaultV2.sol#L450](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L450)
- [ReaperVaultV2.sol#L505](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L505)
- [ReaperVaultV2.sol#L520](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L520)
- [ReaperVaultV2.sol#L521](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L521)
- [ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)
- [ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)
- [ReaperVaultV2.sol#L395](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L395)
- [ReaperVaultV2.sol#L396](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L396)
- [ReaperVaultV2.sol#L444](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L444)
- [ReaperVaultV2.sol#L445](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L445)
- [ReaperVaultV2.sol#L451](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L451)
- [ReaperVaultV2.sol#L452](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L452)
- [ReaperVaultV2.sol#L514](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L514)
- [ReaperVaultV2.sol#L515](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L515)


## 6. do not use named return on function signiture when there is return at the end of the function


- [StabilityPool.sol#L511](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L511)
- [StabilityPool.sol#L626](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L626)

- [ReaperVaultERC4626.sol#L29](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L29)
- [ReaperVaultERC4626.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L37)
- [ReaperVaultERC4626.sol#L51](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L51)
- [ReaperVaultERC4626.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L66)
- [ReaperVaultERC4626.sol#L79](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L79)
- [ReaperVaultERC4626.sol#L96](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L96)
- [ReaperVaultERC4626.sol#L122](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L122)
- [ReaperVaultERC4626.sol#L165](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L165)
- [ReaperVaultERC4626.sol#L220](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L220)
- [ReaperVaultERC4626.sol#L240](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L240)

- [ReaperStrategyGranarySupplyOnly.sol#L104](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L104)


## 7. can make the variable outside the loop to save gas

make the variable outside and only give the value to variable inside

`collateral`
- [StabilityPool.sol#L352](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L352)
- [StabilityPool.sol#L398](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L398)
- [StabilityPool.sol#L832](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L832)
- [StabilityPool.sol#L860](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L860)
- [LQTYStaking.sol#L206](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L206)
- [LQTYStaking.sol#L229](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L229)
- [LQTYStaking.sol#L242](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L242)

`amount`
- [StabilityPool.sol#L353](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L353)
- [StabilityPool.sol#L399](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L399)
- [ReaperStrategyGranarySupplyOnly.sol#L120](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L120)


`currentS`
- [StabilityPool.sol#L833](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L833)

`price` and `lowestTrove` and `collMCR` and `ICR`
- [StabilityPool.sol#L861-L864](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L861-L864)

`F_Collateral_Snapshot`
- [LQTYStaking.sol#L208](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L208)

`strategy`
- [ReaperVaultV2.sol#L265](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L265)

5 variables here
- [ReaperVaultV2.sol#L378-386](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L378-386)


## 8. splitting require() statements that use `&&` saves gas

Instead of using operator && on a single require. Using a two require can save more gas.

- [BorrowerOperations.sol#L653](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L653)

- [LUSDToken.sol#L347](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L347)
- [LUSDToken.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L351)

## 9. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

bring the poistion of the first require to last of them. because its the most expensive require
- [CollateralConfig.sol#L51-L54](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L51-L54)

do the requires for L94 and L97 at the start of the function and then carry on with the rest of the function 
- [CollateralConfig.sol#L91-L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L91-L97)

- [ReaperVaultV2.sol#L150-L156](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L150-L156) these be sorted like this

```
        require(_strategy != address(0), "Invalid strategy address");
        require(_feeBPS <= PERCENT_DIVISOR / 5, "Fee cannot be higher than 20 BPS");
        require(address(this) == IStrategy(_strategy).vault(), "Strategy's vault does not match");
        require(address(token) == IStrategy(_strategy).want(), "Strategy's want does not match");
        require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
        require(strategies[_strategy].activation == 0, "Strategy already added");
        require(_allocBPS + totalAllocBPS <= PERCENT_DIVISOR, "Invalid allocBPS value");
```
swap position of the requires
- [ReaperVaultV2.sol#L180-L181](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L180-L181)

swap position of the requires
- [ReaperVaultV2.sol#L321-L322](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L321-L322)

put the L96 require as the last one
- [ReaperBaseStrategyv4.sol#L96-L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L96-L98)


## 10. Avoid a SLOAD optimistically

There is a chance that the first part will be true so the second part doesn’t need to be checked, so it is better to use the part that we have instead of the part that needs to be called.

`redemptionHelper` can be put at the start of require because we already have it so 2 SLOADS can be avoided
- [ActivePool.sol#L332](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L332)
and one here
- [LQTYStaking.sol#L255](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L255)


## 11. use a more recent version of solidity

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath and support of unchecked keyword which gives possibily for lots of gas saves

Use a solidity version of at least 0.8.2 to get compiler automatic inlining

Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads

Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings

Use a solidity version of at least 0.8.10 to have `external` calls skip contract existence checks if the external call has a return value

Use a solidity version of at least 0.8.12 to get string.concat() to be used instead of abi.encodePacked(<str>,<str>)

Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions

Use a solidity version of at least 0.8.15 because the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

Use a solidity version of at least 0.8.17 to  get prevention of the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.


## 12. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

- [ActivePool.sol#L320](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L320)

- [BorrowerOperations.sol#L438](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L438)
- [BorrowerOperations.sol#L455](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L455)
- [BorrowerOperations.sol#L476](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L476)
- [BorrowerOperations.sol#L533](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L533)
- [BorrowerOperations.sol#L537](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L537)
- [BorrowerOperations.sol#L546](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L546)
- [BorrowerOperations.sol#L551](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L551)
- [BorrowerOperations.sol#L555](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L555)
- [BorrowerOperations.sol#L567](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L567)
- [BorrowerOperations.sol#L571](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L571)
- [BorrowerOperations.sol#L624](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L624)
- [BorrowerOperations.sol#L636](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L636)
- [BorrowerOperations.sol#L661](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L661)
- [BorrowerOperations.sol#L682](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L682)

- [StabilityPool.sol#L439](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L439)
- [StabilityPool.sol#L637](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637)
- [StabilityPool.sol#L693](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L693)
- [StabilityPool.sol#L776](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L776)
- [StabilityPool.sol#L794](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L794)
- [StabilityPool.sol#L869](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L869)

- [ReaperVaultV2.sol#L462](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L462)


## 13. avoid an unnecessary sstore by not writing a default value for bools (ones not caught by c4udit)
there is no need give them the value `false` because they are already false

`initialized`
- [CollateralConfig.sol#L18](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L18)
- [CommunityIssuance.sol#L21](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21)

`addressesSet`
- [ActivePool.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32)

`mintingPaused`
- [LUSDToken.sol#L37](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37)


## 14. Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

these strings can be put into bytes32 instead to save gas

- [ActivePool.sol#L30](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L30)

- [StabilityPool.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L150)

- [LQTYStaking.sol#L23](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L23)

- [LUSDToken.sol#L32](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L32)
- [LUSDToken.sol#L33](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L33)
- [LUSDToken.sol#L34](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L34)


## 15. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [BorrowerOperations.sol#L446-L450](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L446-L450)
- [BorrowerOperations.sol#L490-L494](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L490-L494)
- [BorrowerOperations.sol#L496-L501](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L496-L501)
- [BorrowerOperations.sol#L649-L655](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L649-L655)

- [LUSDToken.sol#L255-L259](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L255-L259)

- [ReaperVaultV2.sol#L331-L335](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L331-L335)
- [ReaperVaultV2.sol#L534-L538](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L534-L538)
- [ReaperVaultV2.sol#L604-L608](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L604-L608)

- [ReaperStrategyGranarySupplyOnly.sol#L92-L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L92-L97)


## 16. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [ReaperVaultV2.sol#L295](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L295)
- [ReaperVaultV2.sol#L278](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L278)


## 17. state var is defined but not used anywhere 

even if they are for readability, consider making them comments instead

`lqtyTokenAddress` is not used anywhere in the contract. remove it to save gas
- [StabilityPool.sol#L160](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L160)


## 18. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 19. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [CollateralConfig.sol#L66](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L66)
- [CollateralConfig.sol#L69](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L69)
- [CollateralConfig.sol#L94](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L94)
- [CollateralConfig.sol#L97](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L97)

11 instances in
- [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol)

- [StabilityPool.sol#L865](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L865)

- [LUSDToken.sol#L282](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L282)

11 instances in 
- [ReaperVaultV2.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol)

- [ReaperVaultERC4626.sol#L80](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L80)
- [ReaperVaultERC4626.sol#L123](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L123)

- [ReaperBaseStrategyv4.sol#L98](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L98)


## 20. instead of assigning to zero use `delete` 

assigning to zero uses more gas than using `delete` , and they both assign variable to default value. so it is encouraged if the data is no longer needed use delete instead.

- [StabilityPool.sol#L529](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L529)

- [ReaperVaultV2.sol#L215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215)
- [ReaperVaultV2.sol#L537](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L537)


## 21. precalculating part of code will reduce the number of operations being done

`(DEGRADATION_COEFFICIENT * 46) / 10**6` is made of constants, so just pre calculate the result and put the answer instead. put `(DEGRADATION_COEFFICIENT * 46) / 10**6` in a comment to show how its calculated to not lower readability of the code
- [ReaperVaultV2.sol#L125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)


## 22. use argument instead of state var

using the argument is way cheaper than doing a SLOAD

- [ReaperVaultV2.sol#L581](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L581)

instead of `want` and `vault` use `_want` and `_vault`

- [ReaperBaseStrategyv4.sol#L74](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L74)

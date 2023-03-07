## 1. Use a more recent version of solidity

The current version being used is `0.6.11`. However, it is considered best practice to use the latest version of solidity. More recent versions of solidity have compiler optimizations, bugfixes, among other things. This could help in reading and writing safe and clean code.

For a list of new releases: [blog.soliditylang.org/category/releases/](https://blog.soliditylang.org/category/releases/)

The following contracts use an old version of solidity:

- [`CollateralConfig.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L3)
- [`BorrowerOperations.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L3)
- [`TroveManager.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L3)
- [`ActivePool.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L3)
- [`StabilityPool.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L3)
- [`CommunityIssuance.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L3)
- [`LQTYStaking.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L3)
- [`LUSDToken.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L3)

## 2. Limit line length

To comply with the [Solidity Style Guide](https://docs.soliditylang.org/en/develop/style-guide.html#maximum-line-length) and increase readability, lines should be limited to **120 characters** max.

Here are some examples:

- File: `TroveManager.sol` [Line 351](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351)
- File: `TroveManager.sol` [Line 393](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393)
- File: `BorrowerOperations.sol` [Line 268](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268)
- File: `BorrowerOperations.sol` [Line 343](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343)
- File: `ActivePool.sol` [Line 269](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L269)

## 3. Use of `uint` instead of `uint256`

`uint` is a shorthand for `uint256`. Making the size of the data explicit reminds the developer and the reader how much data can be stored, which may help prevent or detect bugs.

Here are a few instances of this issue:

File: [BorrowerOperations.sol](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L55)

```solidity
contracts/BorrowerOperations.sol

55:        uint debt;
56:        uint coll;
57:        uint oldICR;
58:        uint newICR;
59:        uint newTCR;
60:        uint LUSDFee;
61:        uint newDebt;
62:        uint newColl;
63:        uint stake;

70:        uint price;
71:        uint LUSDFee;
72:        uint netDebt;
73:        uint compositeDebt;
74:        uint ICR;
75:        uint NICR;
76:        uint stake;
77:        uint arrayIndex;

172:    function openTrove(address _collateral, uint _collAmount, uint _maxFeePercentage, uint _LUSDAmount, address _upperHint, address _lowerHint) external override {
```

## 4. Typo mistakes

It is recommended to use correct spelling/grammar to improve readability.

File: `TroveManager.sol` [Line 936](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L936)

```
    /// @audit (elswhere --> elsewhere), (zero'd --> zeroed)
    * The debt recorded on the trove's struct is zero'd elswhere, in _closeTrove.
```

## 5. Use named imports rather than unnamed imports

Using named imports (`import {x} from "y.sol"`) will speed up compilation, and avoids polluting the namespace making flattened files smaller.

This issue is present in every file in scope.

## 6. `CCR` can be smaller than `MCR`

In `CollateralConfig.sol`, the `initialize()` function never ensures that the `MCR` cannot be greater than `CCR`. Consider adding a sanity check to avoid accidents.

For example:

File: `CollateralConfig.sol` [Line 46-76](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L46-L76)

```solidity
46:    function initialize(
47:        address[] calldata _collaterals,
48:        uint256[] calldata _MCRs,
49:        uint256[] calldata _CCRs
50:    ) external override onlyOwner {
...
66:            require(_MCRs[i] >= MIN_ALLOWED_MCR, "MCR below allowed minimum");
67:            config.MCR = _MCRs[i];
68:
69:            require(_CCRs[i] >= MIN_ALLOWED_CCR, "CCR below allowed minimum");
70:            config.CCR = _CCRs[i];
71:
72:            emit CollateralWhitelisted(collateral, decimals, _MCRs[i], _CCRs[i]);
...
76:    }
```

Consider adding the following `require()` to the above function:

`require(_CCRs[i] >= _MCRs[i], "MCR cannot be greater than CCR");`

## 7. Contracts are not fully documented with NatSpec

It is recommended that contracts be thoroughly documented using [NatSpec](https://docs.soliditylang.org/en/develop/natspec-format.html). Using NatSpec will improve readability for all readers. However, most contracts do not utilize it fully.

For example, this following contract is missing NatSpec fully:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

## 8. Unspecific Compiler Version Pragma

Floating pragmas should be avoided for non-library contracts as it may be a security risk.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

For example, `pragma solidity ^0.8.0;` is very unspecific.

- [`ReaperVaultV2.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3)
- [`ReaperVaultERC4626.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3)
- [`ReaperBaseStrategyv4.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3)
- [`ReaperStrategyGranarySupplyOnly.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3)

## 9. Unused named returns

Using both named returns and a return statement is unnecessary. Removing one of those can improve code clarity.

For example, in the following function, the named return (`uint index`) is unused:

File: `TroveManager.sol` [Line 1316-1319](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1316-L1319)

```solidity
    function addTroveOwnerToArray(address _borrower, address _collateral) external override returns (uint index) {
        _requireCallerIsBorrowerOperations();
        return _addTroveOwnerToArray(_borrower, _collateral);
    }
```

Consider either removing the unused named returns or using it instead.

Here is 1 other instance of this issue:

- File: `TroveManager.sol` [Line 1321-1333](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321-L1333)

## 10. Use time units for better readability

[Solidity has time units](https://docs.soliditylang.org/en/v0.8.14/units-and-global-variables.html) (seconds, minutes, hours, etc). As such, consider using time units instead of using literal numbers, this will increase readability and decrease the likelihood of mistakes being made.

For example, the following `constant` could use time units:

File: `TroveManager.sol` [Line 48](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Core/contracts/TroveManager.sol#L48)

```
    uint constant public SECONDS_IN_ONE_MINUTE = 60;
```

## 11. `constant` variables should be defined rather than using magic numbers

In the following instances, consider declaring a `constant` instead of magic numbers:

File: `ReaperVaultV2.sol` [Line 125](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L125)

```solidity
File: contracts/ReaperVaultV2.sol

125: lockedProfitDegradation = (DEGRADATION_COEFFICIENT * 46) / 10**6; // 6 hours in blocks
```

File: [`ActivePool.sol`](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L127)

```solidity
File: contracts/ActivePool.sol

        /// @audit 10_000
127:    require(_bps <= 10_000, "Invalid BPS value");

        /// @audit 500
133:    require(_driftBps <= 500, "Exceeds max allowed value of 500 BPS");

        /// @audit 10_000
145:    require(_treasurySplit + _SPSplit + _stakingSplit == 10_000, "Splits must add up to 10000 BPS");

        /// @audit 10_000
258:    vars.percentOfFinalBal = vars.finalBalance == 0 ? uint256(-1) : vars.currentAllocated.mul(10_000).div(vars.finalBalance);

        /// @audit 10_000
262:    vars.finalYieldingAmount = vars.finalBalance.mul(vars.yieldingPercentage).div(10_000);
```

## 12. Large numbers are hard to read

Consider using underscores/scientific notation on large numbers to increase readability.

For example:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L41

```diff
-    uint256 public constant PERCENT_DIVISOR = 10000;
+    uint256 public constant PERCENT_DIVISOR = 10_000;
```

Here is 1 other instance of this issue:

- File: `TroveManager.sol` [Line 53](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L53)

## 13. Use `delete` to clear variables instead of zero assignment, i.e. (0, 0x0, false)

A better way to indicate that you are clearing a variable is to use the [`delete`](https://docs.soliditylang.org/en/v0.8.17/types.html#delete) keyword.

For example:

File: `ReaperVaultV2.sol` [Line 215](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L215)

```solidity
        strategies[_strategy].allocBPS = 0;
```

can be changed to:

```solidity
        delete strategies[_strategy].allocBPS;
```

Here are some other instances of this issue:

File: `TroveManager.sol` ([387](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L387), [388](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L388), [468](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L468), [469](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L469), [505](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L505), [506](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L506), [1192](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1192), [1285](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1285), [1286](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1286), [1288](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1288), [1289](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1289))

```solidity
387:        singleLiquidation.debtToOffset = 0;
388:        singleLiquidation.collToSendToSP = 0;

468:        debtToOffset = 0;
469:        collToSendToSP = 0;

505:        singleLiquidation.debtToRedistribute = 0;
506:        singleLiquidation.collToRedistribute = 0;

1192:       Troves[_borrower][_collateral].stake = 0;

1285:       Troves[_borrower][_collateral].coll = 0;
1286:       Troves[_borrower][_collateral].debt = 0;

1288:       rewardSnapshots[_borrower][_collateral].collAmount = 0;
1289:       rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
```

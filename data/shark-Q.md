## 1. Use a more recent version of solidity

The current version being used is `0.6.11`. However, it is considered best practice to use the latest version of solidity. More recent versions of solidity have compiler optimizations, and bugfixes, among other things. This could help in reading and writing safe and clean code.

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

File: `BorrowerOperations.sol` [Line 127](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L127)

```
        /// @audit "This makes impossible" --> "This makes it impossible"
        // This makes impossible to open a trove with zero withdrawn LUSD
```

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

For example, the following contract is missing NatSpec fully:

https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol

## 8. Unspecific Compiler Version Pragma

Floating pragmas should be avoided for non-library contracts as they may be a security risk.

A known vulnerable compiler version may accidentally be selected or security tools might fall back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

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

Consider either removing the unused named returns or using them instead.

Here is 1 other instance of this issue:

- File: `TroveManager.sol` [Line 1321-1333](https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1321-L1333)

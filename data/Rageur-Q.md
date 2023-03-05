## LOW-1: State variable visibility not set

### Code

address stabilityPoolAddress;
address gasPoolAddress;
ICollSurplusPool collSurplusPool;

### Affected file

* BorrowerOperations.sol (Line: 28, 30, 32)
* TroveManager.sol (Line: 31, 33)

## LOW-2: Floating Pragma

### Code

pragma solidity ^0.8.0;

### Affected file

* ReaperBaseStrategyv4.sol Line(3)
* ReaperStrategyGranarySupplyOnly.sol Line(3)
* ReaperVaultERC4626.sol Line(3)
* ReaperVaultV2.sol Line(3)

## LOW-3: Outdated compiler version

### Code

pragma solidity 0.6.11;

### Affected file

* ActivePool.sol Line(3)
* BorrowerOperations.sol Line(3)
* CollateralConfig.sol Line(3)
* CommunityIssuance.sol Line(3)
* LQTYStaking.sol Line(3)
* LUSDToken.sol Line(3)
* StabilityPool.sol Line(3)
* TroveManager.sol Line(3)
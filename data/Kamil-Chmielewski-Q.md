# QA Report
### Non-Critical Issues

| # | Title | Type | Instances |
| :--: | --- | :---: | :---: |
| N-01 | Use named import syntax | Code Quality | All contracts in scope | 

### [N-01] Use named import syntax
Instead of importing entire files, use named import syntax. As the repository grows, static analyzers like Slither or the Solidity compiler might raise "duplicate definition" errors because some dependencies might have overlapping names.

Additionally, using explicit import names increases readability. In the example below in the `CollateralConfig.sol` contract, it is immediately visible that the `IERC20` is imported from the `SafeERC20.sol` thanks to the named import syntax.

```diff
--- a/Ethos-Core/contracts/CollateralConfig.sol
+++ b/Ethos-Core/contracts/CollateralConfig.sol
@@ -2,10 +2,10 @@
 
 pragma solidity 0.6.11;
 
-import "./Dependencies/CheckContract.sol";
-import "./Dependencies/Ownable.sol";
-import "./Dependencies/SafeERC20.sol";
-import "./Interfaces/ICollateralConfig.sol";
+import {CheckContract} from "./Dependencies/CheckContract.sol";
+import {Ownable} from "./Dependencies/Ownable.sol";
+import {SafeERC20, IERC20} from "./Dependencies/SafeERC20.sol";
+import {ICollateralConfig} from "./Interfaces/ICollateralConfig.sol";
```


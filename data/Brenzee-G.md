## No need to specify value if that value is the initial value of the type
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/CollateralConfig.sol#L18
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L32
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/CommunityIssuance.sol#L21
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L37

### Explanation

By default `bool` values in Solidity are `false` . By removing the `= false` gas can be saved.

CollateralConfig.sol:
Deploy gas before: 871266
Deploy gas after: 870858
Gas saved: 408

ActivePool.sol
Deploy gas before: 2391096
Deploy gas after: 2390677
Gas saved: 419

Average saved gas per smart contract deployement - 410 gas

## Suggestions

```diff
diff --git a/contracts/CollateralConfig.sol.orig b/contracts/CollateralConfig.sol
index d524f23..6ad85b1 100644
--- a/contracts/CollateralConfig.sol.orig
+++ b/contracts/CollateralConfig.sol
@@ -15,7 +15,7 @@ import "./Interfaces/ICollateralConfig.sol";
 contract CollateralConfig is ICollateralConfig, CheckContract, Ownable {
     using SafeERC20 for IERC20;

-    bool public initialized = false;
+    bool public initialized;

     // Smallest allowed value for the minimum collateral ratio for individual troves in each market (collateral)
     uint256 constant public MIN_ALLOWED_MCR = 1.1 ether; // 110%
```

```diff
diff --git a/contracts/ActivePool.sol.orig b/contracts/ActivePool.sol
index 753fcd0..868a35e 100644
--- a/contracts/ActivePool.sol.orig
+++ b/contracts/ActivePool.sol
@@ -29,7 +29,7 @@ contract ActivePool is Ownable, CheckContract, IActivePool {

     string constant public NAME = "ActivePool";

-    bool public addressesSet = false;
+    bool public addressesSet;
     address public collateralConfigAddress;
     address public borrowerOperationsAddress;
     address public troveManagerAddress;
```

```diff
diff --git a/contracts/LQTY/CommunityIssuance.sol.orig b/contracts/LQTY/CommunityIssuance.sol
index c79adfe..6a619df 100644
--- a/contracts/LQTY/CommunityIssuance.sol.orig
+++ b/contracts/LQTY/CommunityIssuance.sol
@@ -18,7 +18,7 @@ contract CommunityIssuance is ICommunityIssuance, Ownable, CheckContract, BaseMa

     string constant public NAME = "CommunityIssuance";

-    bool public initialized = false;
+    bool public initialized;

    /* The issuance factor F determines the curvature of the issuance curve.
     *
```

```diff
diff --git a/contracts/LUSDToken.sol.orig b/contracts/LUSDToken.sol
index 9c5424d..dd470c4 100644
--- a/contracts/LUSDToken.sol.orig
+++ b/contracts/LUSDToken.sol
@@ -34,7 +34,7 @@ contract LUSDToken is CheckContract, ILUSDToken {
     string constant internal _VERSION = "1";
     uint8 constant internal _DECIMALS = 18;

-    bool public mintingPaused = false;
+    bool public mintingPaused;

     // --- Data for EIP2612 ---
```
## Use `require()` instead of `assert()`
BorrowerOperations.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L128
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L197
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L301
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L331

LUSDToken.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L312-L313
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L321
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L329
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337-L338

RedemptionHelper.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L337-L338

StabilityPool.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L526
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L551
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L591

TroveManager.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L417
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1224
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1279
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1342-L1348
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1414
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1489


### Explanation

The Solidity `assert()` function is meant to assert invariants and `require()` analyses conditions. When condition inside the `assert` is false, it tends to consume all of the remaining gas, but when condition inside `require` is false it refunds all the remaining gas of the transaction `gasLimit`

Example of BorrowerOperations.sol
```diff
diff --git a/contracts/BorrowerOperations.sol.orig b/contracts/BorrowerOperations.sol
index a743611..43cf88e 100644
--- a/contracts/BorrowerOperations.sol.orig
+++ b/contracts/BorrowerOperations.sol
@@ -125,7 +125,7 @@ contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOpe
         onlyOwner
     {
         // This makes impossible to open a trove with zero withdrawn LUSD
-        assert(MIN_NET_DEBT > 0);
+        require(MIN_NET_DEBT > 0);

         checkContract(_collateralConfigAddress);
         checkContract(_troveManagerAddress);
@@ -194,7 +194,7 @@ contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOpe

         // ICR is based on the composite debt, i.e. the requested LUSD amount + LUSD borrowing fee + LUSD gas comp.
         vars.compositeDebt = _getCompositeDebt(vars.netDebt);
-        assert(vars.compositeDebt > 0);
+        require(vars.compositeDebt > 0);

         vars.ICR = LiquityMath._computeCR(_collAmount, vars.compositeDebt, vars.price, vars.collDecimals);
         vars.NICR = LiquityMath._computeNominalCR(_collAmount, vars.compositeDebt, vars.collDecimals);
@@ -298,7 +298,7 @@ contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOpe
         _requireTroveisActive(contractsCache.troveManager, _borrower, _collateral);

         // Confirm the operation is either a borrower adjusting their own trove, or a pure collateral transfer from the Stability Pool to a trove
-        assert(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));
+        require(msg.sender == _borrower || (msg.sender == stabilityPoolAddress && _collTopUp > 0 && _LUSDChange == 0));

         contractsCache.troveManager.applyPendingRewards(_borrower, _collateral);

@@ -328,7 +328,7 @@ contract BorrowerOperations is LiquityBase, Ownable, CheckContract, IBorrowerOpe
             vars.price,
             vars.collDecimals
         );
-        assert(_collWithdrawal <= vars.coll);
+        require(_collWithdrawal <= vars.coll);

         // Check the adjustment satisfies all conditions for the current system mode
         _requireValidAdjustmentInCurrentMode(isRecoveryMode, _collateral, _collWithdrawal, _isDebtIncrease, vars);
```

### Recommendation
Change `assert()` to `require()` 

## Inconsistent `uint` and `uint256` usage
TroveManager.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L129-L138
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L146-L158

BorrowerOperations.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L47-L78

### Explanation
`uint` and `uint256` are the same types. Mixing `uint256` and `uint` is not a good code style.

### Recommendation
Either use `uint256` or `uint` across all of the contracts.

## Call to `keccak256()` should use `immutable` rather than `constant`
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76

### Recommendation
Change `constant` to `immutable`

```solidity
bytes32 public constant DEPOSITOR = keccak256("DEPOSITOR");
bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
bytes32 public constant ADMIN = keccak256("ADMIN");

bytes32 public immutable DEPOSITOR = keccak256("DEPOSITOR");
bytes32 public immutable STRATEGIST = keccak256("STRATEGIST");
bytes32 public immutable GUARDIAN = keccak256("GUARDIAN");
bytes32 public immutable ADMIN = keccak256("ADMIN");
```

## Avoid setting time variables with integer values
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L48

```diff
diff --git a/contracts/TroveManager.sol.orig b/contracts/TroveManager.sol
index 6f587d9..d93899a 100644
--- a/contracts/TroveManager.sol.orig
+++ b/contracts/TroveManager.sol
@@ -45,7 +45,7 @@ contract TroveManager is LiquityBase, /*Ownable,*/ CheckContract, ITroveManager

     // --- Data structures ---

-    uint constant public SECONDS_IN_ONE_MINUTE = 60;
+    uint constant public SECONDS_IN_ONE_MINUTE = 1 minutes;
     /*
      * Half-life of 12h. 12h = 720 min
      * (1/2) = d^720 => d = (1/2)^(1/720);
```

### Recommendation
It is better to use `minutes`, `hours`, `days` etc. It makes the code more readable and easier to spot issues related to time.
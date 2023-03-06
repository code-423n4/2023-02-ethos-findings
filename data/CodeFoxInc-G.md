# Gas Optimization
## Gas-01 Not necessary to reinitiate the default value in struct

In the function `addStrategy()`. It is not necessary to initiate the value as 0. So we can remove this part to save gas. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L158-L166](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L158-L166)

```diff
-	strategies[_strategy] = StrategyParams({ 
-	     activation: block.timestamp,
-            feeBPS: _feeBPS,
-            allocBPS: _allocBPS,
-            allocated: 0,
-            gains: 0,
-            losses: 0,
-            lastReport: block.timestamp
-       });
+       strategies[_strategy].activation = block.timestamp;
+       strategies[_strategy].feeBPS = _feeBPS;
+       strategies[_strategy].allocBPS = _allocBPS;
+       strategies[_strategy].lastReport = block.timestamp; 
```

### Proof of Concept

Before: 

```
····································|······························|·············|·············|·············|···············|··············
|  ReaperVaultERC4626               ·  addStrategy                 ·     170982  ·     205182  ·     182382  ·            3  ·          -  │
····································|······························|·············|·············|·············|···············|··············
```

After: 

```
····································|······························|·············|·············|·············|···············|··············
|  ReaperVaultERC4626               ·  addStrategy                 ·     164169  ·     198369  ·     175569  ·            3  ·          -  │
····································|······························|·············|·············|·············|···············|··············
```

### Recommendation

As a result it generally save about 7,000 gas. So I recommend that you change this part as code snippet showed above. 

## Gas-02 Change the sequence of the state variables for storage optimization

Put the bool state variable at the last so that the address and bool state variable can be set into one storage slot. This can save storages’ gas consumption. 

### Code Snippet

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L49](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L49)

### Recommendation

```diff
-	 bool public emergencyShutdown;
		
		...
          bytes32 public constant STRATEGIST = keccak256("STRATEGIST");
          bytes32 public constant GUARDIAN = keccak256("GUARDIAN");
          bytes32 public constant ADMIN = keccak256("ADMIN");

          address public treasury; // address to whom performance fee is remitted in the form of vault shares
+        bool public emergencyShutdown;
```

## Gas-03 Use `delete` instead of `=0` to reset the value

Use `delete` to reset the value instead of setting it to zero. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollSurplusPool.sol#L99](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollSurplusPool.sol#L99)

```diff
-	balances[_account][_collateral] = 0; 
+	delete balances[_account][_collateral]; 
        emit CollBalanceUpdated(_account, _collateral, 0); // @audit use delete to reset the value
```

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1288-L1289](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/TroveManager.sol#L1288-L1289)

```diff
-	Troves[_borrower][_collateral].coll = 0;
-       Troves[_borrower][_collateral].debt = 0;
+	delete Troves[_borrower][_collateral].coll;
+      delete Troves[_borrower][_collateral].debt;
-	rewardSnapshots[_borrower][_collateral].collAmount = 0;
-       rewardSnapshots[_borrower][_collateral].LUSDDebt = 0;
+	delete rewardSnapshots[_borrower][_collateral].collAmount;
+      delete rewardSnapshots[_borrower][_collateral].LUSDDebt;
```

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L215](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L215)

```diff
-     strategies[_strategy].allocBPS = 0;
+    delete strategies[_strategy].allocBPS;
```

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L369-L371](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L369-L371)

```diff
-	    uint256 totalLoss = 0;
+          delete uint256 totalLoss;
            uint256 queueLength = withdrawalQueue.length;
-           uint256 vaultBalance = 0;
+          delete uint256 vaultBalance;
```

## Gas-04.  Use `x = x + y` instead of `+=`

Use `x = x + y` , `x = x - y`instead of `x += y` , `x -= y` to save gas. 

### Code snippet

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L196](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L196)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L391](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L391)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L450](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L450)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L505](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L505)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L520-L521](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L520-L521)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L194](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L194)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L214](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L214)’

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L390-L396](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L390-L396)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L453](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L444-L453)

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L514-L516](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L514-L516)

## Gas-05. Change `storage` to `memory`

Change `storage` to `memory` to save gas. 

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L266](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L266)

### Recommendation

```diff
-    StrategyParams storage params = strategies[strategy];
+   StrategyParams memory params = strategies[strategy];
```

## Gas-06. Set the function as `pure`

Set the function as `pure` to save gas. 

### Code snippet

[https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L659](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L659)

[https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203](https://github.com/code-423n4/2023-02-ethos/blob/d46ede81e35c1ed86242c9ba3c5c57d2ca2b57d8/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L203)
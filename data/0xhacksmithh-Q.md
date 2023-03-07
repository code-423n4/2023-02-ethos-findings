### [Low-01]```approve``` could be front-runned
*instances()*
```
File: Ethos-Core/contracts/LUSDToken.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L230
```


### [NC-01] OMISSIONS In Events
*instances(Multiple Instances as shown in folling links)*
```
File: Ethos-Core/contracts/BorrowerOperations.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L92-L106
```
```
File: Ethos-Core/contracts/TroveManager.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L186-L223
```
```
File: Ethos-Core/contracts/ActivePool.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/ActivePool.sol#L58-L67
```
```
File: Ethos-Core/contracts/StabilityPool.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L233-L257
```
```
File: Ethos-Core/contracts/LUSDToken.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L78-L82
```
```
File: Ethos-Core/contracts/LQTY/LQTYStaking.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LQTY/LQTYStaking.sol#L49-L62
```

### [NC-02] No ```Natspec``` for external or public fnctions
*instances(6)*
```
File: Ethos-Core/contracts/TroveManager.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L310
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L513
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L939
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L962
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L982
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1016
```

### [NC-03] ```constant``` values such as a call to ```keccake()```, should use ```imutable``` rather than ```constant```
*instances(9)*
```
File: Ethos-Core/contracts/LUSDToken.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L42
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L44
```
```
File: Ethos-Vault/contracts/ReaperVaultV2.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L73-L76
```
```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L49-L52
```

### [NC-04] ```ecrecover``` could lead to Signature Malibility
*instances(1)*
```
File: Ethos-Core/contracts/LUSDToken.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/LUSDToken.sol#L287
```

### [NC-05] Hardcoded Value
*instances(3)*
```
File: Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L24-L29
```

### [NC-06] Unused Code should be removed from code base
*instances()*
```
File: Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol
https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L144
```
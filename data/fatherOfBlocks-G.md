**Ethos-Core/contracts/BorrowerOperations.sol**
- L347/356/433/435/468/470/473/542/543/547/548/677/678/699/670/744/745 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

- L525/529/530/534/538/543/552/563/568/617/621/625/629/633/637/641/645/651/654 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTE COST EXTRA GAS


**Ethos-Core/contracts/TroveManager.sol**
- L343/348/420/421/819/828/1048/1049/1050/1061/1062/1063/1067/1068/1070/1071/1073/1124/1129/1131/1132/1139/1140/1144/1147 /1190/1203/1206/1309/1310/1362/1363/1367/1368/1369/1384/1386/1501/1503/1510/1511/1513 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


**Ethos-Core/contracts/ActivePool.sol**
- L107/133/324/334/341 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

- L328/332 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


**Ethos-Core/contracts/StabilityPool.sol**
- L687/688/690/707/709 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.


**Ethos-Core/contracts/LQTY/CommunityIssuance.sol**
- L107/108/109 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

- L133 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**Ethos-Core/contracts/LQTY/LQTYStaking.sol**
- L208/209/218/219/220/252/255 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

- L257/262/266/270 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**Ethos-Core/contracts/LUSDToken.sol**
- L136/315/331/350/356/362/371/377/386/390/394 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**Ethos-Vault/contracts/ReaperVaultV2.sol**
- L150/321/436/619 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS

- L323/324/326/329/330/334/384/385/435/436/466/467 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

- L384 - A subtraction is performed, which is checked in L374, therefore an unchecked operation could be performed and no unnecessary extra validation is performed.

- L629/639 - A validation is performed after executing _atLeastRole() and there is no direct relationship between them, therefore the validation should be performed before, on line 628/638.


**Ethos-Vault/contracts/abstract/ReaperBaseStrategyV4.sol**
- L98/193 - REQUIRE()/REVERT() STRINGS LONGER THAN 32 BYTES COST EXTRA GAS


**Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol**
- L81/93/100/132/136 - A subtraction is performed, which in L80/92/99/131/135 is checked, therefore an unchecked operation could be performed and no unnecessary extra validation is performed.

- L119/120/198/199 - A variable is created in memory that will only be used once, therefore it could be avoided and used directly where the operation is necessary.

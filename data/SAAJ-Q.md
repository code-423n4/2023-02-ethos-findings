
# Low Risk Issues

Some of the opportunities identified for improving low severity issues throughout the codebase of Ethos Reserve are categorised into 04 main areas; with further multiple instances in each of the category.


# [L-01] Floating of Pragma (04 Instances)
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
1.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultV2.sol#L3
2.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperVaultERC4626.sol#L3
3.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L3
4.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/contracts/ReaperStrategyGranarySupplyOnly.sol#L3

# [L 02] Using obsolete dependency of openzeppelin (04 Instances)
Package.json configuration is using 4.7.3 version which is not updated. Latest non-vulnerable version should be use instead to avoid any bug.
Link to the code:
1.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L41
2.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Vault/package.json#L42
3.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L34
4.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/package.json#L35

# [L 03] Long lines are not suitable for Solidity Style Guide (27 Instances)
It is recommended that lines in the source code should not exceed 80-120 characters. Code lines should be split into multiline output when they reach that length.

Link to the code:
1.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L105
2.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L172
3.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L235
4.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L248
5.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L268
6.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L282
7.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L343
8.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L419
9.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L511
10.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L200
11.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L201
12.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L348
13.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L351
14.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L393
15.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L398
16.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L404
17.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L407
18.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L421
19.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L428
20.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L572
21.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L575
22.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L915
23.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L926
24.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1258
25.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/TroveManager.sol#L1259
26.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L637
27.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/StabilityPool.sol#L657


# [L-04] Minting or burning tokens to the zero address should be avoided (02 Instances)

The functions are based on issuance or burning of LUSD which can affect total active debt.


Address(0) check is missing in both functions, consider applying check to ensure tokens aren’t minted or burned to the zero address that will directly impact total debt.


Link to the code:

1.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L513
2.	https://github.com/code-423n4/2023-02-ethos/blob/main/Ethos-Core/contracts/BorrowerOperations.sol#L519

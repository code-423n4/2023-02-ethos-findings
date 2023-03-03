# Lock pragma version

## Proof of Concept

A few of the contracts within the scope of the contest have a floating pragma:
- Ethos-Vault/contracts/ReaperVaultV2.sol
- Ethos-Vault/contracts/ReaperVaultERC4626.sol
- Ethos-Vault/contracts/ReparerBaseStrategyv4.sol

```
pragma solidity ^0.8.0;
```

CWE: https://swcregistry.io/docs/SWC-103

## Impact

With a floating pragma, contracts might be compiled with a compiler version that has not been tested and might introduce unexpected bugs.

## Mitigation suggestion

Lock pragma to a specific version, e.g. `pragma solidity 0.8.17;`.
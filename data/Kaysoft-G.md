## [G-01] USE {unchecked ++i}/{unchecked i++} for ++i/i++ when operation is not possible to overflow.

Files:
- [Ethos-Core/contracts/CollateralConfig.sol#L56](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/CollateralConfig.sol#L56)

- [Ethos-Core/contracts/StabilityPool.sol#L351](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L351)

- [Ethos-Core/contracts/StabilityPool.sol#L397](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L397)

- [Ethos-Core/contracts/StabilityPool.sol#L640](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L640)

- [Ethos-Core/contracts/StabilityPool.sol#L810](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L810)

- [Ethos-Core/contracts/StabilityPool.sol#L831](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L831)

- [Ethos-Core/contracts/StabilityPool.sol#L859](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L859)

## [G-02] require()/revert() strings that are longer that 32 bytes cost extra gas

Every extra bytes more than the original 32 incurs extra MSTORE which is 3 gas.
see: https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#consider-having-short-revert-strings

Files:
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L150](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L150)

```
require(!emergencyShutdown, "Cannot add strategy during emergency shutdown");
```

## [G-03]  x = x + y cost less gas than x +=y for state variables. Same for x = x -y.

File:
- [Ethos-Vault/contracts/ReaperVaultV2.sol#L168](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L168)
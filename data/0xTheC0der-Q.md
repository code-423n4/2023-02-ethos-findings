# Low severtity findings

## Vault roles - centralization risk
Require that the addresses in `_strategists[]` and `_multisigRoles[]` in [ReaperBaseStrategyv4 .__ReaperBaseStrategy_init](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/abstract/ReaperBaseStrategyv4.sol#L63-L88) as well as [ReaperVaultV2.constructor](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136) are all different from each other.

## Wrong NatSpec comment about decimals
According to the NatSpect comment of [ReaperVaultV2.getPricePerFullShare](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L291-L297), the functions returns an uint256 with 18 decimals. However, the code shows that the underlying token's decimals are mirrored which might be != 18. This function is not called by any other contract under scope, but an external contract might be at risk when relying on the wrong comment.

## Limited upgradeability of `ReaperStrategyGranarySupplyOnly` contract
The upgradeable contract `ReaperStrategyGranarySupplyOnly` inherits from the non-stateless contract `ReaperBaseStrategyv4` which does not contain a storage gap (e.g. `uint256[50] private __gap;`) after its storage variables. Therfore, adding another storage variable to `ReaperBaseStrategyv4` will shift down the storage in the inheritance chain. This will break the `ReaperStrategyGranarySupplyOnly` contract in case of an upgrade. Consider adding a gap to `ReaperBaseStrategyv4` in order to reserve space for future storage variables.

# Non-critical findings

## Vault depositor role not granted to anyone
The vault [DEPOSITOR](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L73) role is not granted to anyone within the whole codebase. I suppose the role is granted on demand by the default admin after deployment. However, you might also want to handle this within [ReaperVaultV2.constructor](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Vault/contracts/ReaperVaultV2.sol#L111-L136) like you do with the other roles.

## ToDo's in `StabilityPool` contract
There are two ToDo's ([#1](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L335-L339) and [#2](https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/StabilityPool.sol#L380-L384)) about unused local variables in the `StabilityPool` contract.
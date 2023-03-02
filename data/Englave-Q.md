L01. LUSDToken has no possibility to remove privileged roles
`upgradeProtocol` function allows adding new `troveManagers`, `stabilityPools` and `borrowerOperations`, granting them a privileged role in the system, but there is no possibility to revoke such a role.
In case of loss of keys to the account there would be no possibility to remove specific addresses and protect the contract.
Path: ./contracts/LUSDToken.sol : upgradeProtocol()
Recommendation: Add a function to remove previously added addresses from privileged roles.